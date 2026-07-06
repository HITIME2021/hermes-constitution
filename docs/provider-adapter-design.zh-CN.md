# Provider Adapter 设计

Provider Adapter 是 Hermes Workflow Protocol 和具体 Provider CLI/API 之间的边界层。

Hermes Core 只负责状态、策略、路由、审查和记忆。Provider 只负责完成被分配的工作。Adapter 负责把 Hermes 的结构化请求翻译成 Provider 原生命令或 API 调用，并把 Provider 输出归一化回 Hermes 协议对象。

## 目标

- 将 `ExecutionRequest` 转换为 Provider 原生调用。
- 将 Provider 输出归一化为 `ExecutionResult` 或 `ProviderError`。
- 在 Workflow Engine 接收结果之前完成错误分类。
- 强制执行 `allowed_scope`、`forbidden_scope`、风险等级和审批证据。
- 保留足够证据，供 Review Gate、重试策略和 Memory Center 使用。
- 避免 Hermes Core 依赖任何 Provider GUI。

## 非目标

- Adapter 不做架构决策。
- Adapter 不直接修改 Task 状态。
- Adapter 不绕过 Project Policy、Risk Policy 或 Review Gate。
- Adapter 不把 CLI/API 传输失败伪装成语义任务失败。
- Adapter 不把 Provider 的自由文本输出直接当作可信状态。

## 逻辑接口

每个 Adapter 都应暴露同一组逻辑操作：

```text
submit_execution(request) -> AdapterSubmission | ProviderError
get_status(external_execution_id) -> AdapterStatus | ProviderError
collect_result(external_execution_id) -> ExecutionResult | ProviderError
cancel(external_execution_id) -> CancelResult | ProviderError
```

Provider 的真实命令、参数、API payload、鉴权和运行目录必须封装在 Adapter 内部。Hermes Core 不应知道 `codex exec`、CodeBuddy CLI 或未来 Provider 的具体命令形态。

## 提交契约

Adapter 接收的输入必须是通过 JSON Schema 校验的 `ExecutionRequest`。提交前必须检查：

- `assigned_provider` 是否和当前 Adapter 匹配。
- `allowed_scope.files` 是否全部在 Project Policy 允许范围内。
- `forbidden_scope.files` 和 `forbidden_scope.actions` 是否没有被请求触碰。
- `allowed_scope.commands` 是否被 Project Policy 允许。
- medium 及以上风险是否包含 Codex planning 或 Codex review 要求。
- 需要审批的文件、命令或动作是否带有审批证据。
- Provider context 中是否包含 secret、token、私钥或不该下发给低成本执行器的敏感上下文。

成功提交返回：

```json
{
  "execution_id": "exec-001",
  "attempt_id": "attempt-001",
  "provider": "codebuddy",
  "external_execution_id": "codebuddy-run-921",
  "status": "accepted",
  "started_at": "2026-07-03T00:00:00Z"
}
```

无法安全提交时返回 `ProviderError`，通常是 `provider_error.adapter_protocol_mismatch`、`execution_error.scope_conflict` 或 `execution_error.policy_violation`。

## 状态映射

Adapter 必须把 Provider 原生状态映射为 Hermes attempt status：

```text
queued       -> pending
running      -> running
completed    -> succeeded
timeout      -> transport_failed
network      -> transport_failed
rate_limit   -> transport_failed
cli_error    -> transport_failed | provider_failed
tool_error   -> provider_failed
test_failed  -> execution_failed
review_failed -> execution_failed
cancelled    -> cancelled
```

当映射不明显时，Adapter 必须附带原始证据和分类理由。Workflow Engine 不应猜测 Provider 错误语义。

## 错误分类

Adapter 返回错误前必须先分类：

```text
transport_error
  Provider 通道失败，例如 timeout、network_error、rate_limit、auth_error、
  provider_unavailable、cli_crash、malformed_response。

provider_error
  Provider 进程或 Adapter 自身失败，例如 tool_unavailable、
  sandbox_setup_failed、adapter_protocol_mismatch。

execution_error
  被分配工作在语义上失败，例如 test_failure、implementation_failed、
  review_failed、scope_conflict、missing_context、invalid_assumption。
```

传输错误默认不触发任务重规划，只消耗 `ExecutionAttempt` 的重试预算。只有语义执行失败、审查失败、重复实现失败、无效假设或新发现约束才可能触发 replanning。

## 交互式确认边界

Provider CLI 在执行时可能要求交互式确认，例如：

```text
trust directory
allow command
apply patch
continue despite risk
authenticate or log in
install dependency
use network
write outside sandbox
approve elevated permission
```

Hermes 不得自动确认这些提示。除非当前任务已经明确批准了对应的具体动作，否则 Hermes 不得传入 `-y`、`--yes`、`--force`、`--accept` 或等价绕过参数。

如果 Provider CLI 需要确认、需要 stdin/TTY 交互，或返回与确认相关的失败，Adapter 必须停止并返回分类结果：

```yaml
provider_error:
  category: provider_error
  type: needs_user_confirmation
  provider: codex | codebuddy
  retryable: false
  counts_as_task_failure: false
  should_trigger_replanning: false
  recommended_action: ask_operator
```

面向操作者的请求必须包含：

```text
provider
prompt_type
requested_action
risk
exact command or operation, when safe
recommended_response
whether approval would expand scope, permissions, dependencies, or auth state
```

未知确认提示默认视为不安全，直到操作者明确决定。Hermes 不得把确认提示当作普通 transport failure 反复重试。

## Codex Adapter

Codex 是 Hermes 的高级推理 Provider。

默认用途：

- planning
- architecture design
- algorithm design
- risk evaluation
- task decomposition
- code review
- failure analysis
- replanning
- memory synthesis

Codex Adapter 要求：

- 在授权范围内提供全局推理上下文。
- 支持只读审查请求，不要求修改文件。
- 将 planning、review 和 failure analysis 的理由结构化返回。
- 将 CLI/API timeout 分类为 `transport_error.timeout`。
- malformed response 先归为 adapter/provider 问题，不能直接算作任务失败。

## CodeBuddy Adapter

CodeBuddy 是低成本、强边界的 scoped executor。

默认用途：

- file-level implementation
- repetitive edits
- small bug fixes
- test writing
- low-risk refactors

CodeBuddy Adapter 要求：

- 只提交有明确文件、命令和动作边界的 execution packet。
- 显式下发 `allowed_scope` 和 `forbidden_scope`。
- 默认拒绝 auth、security、db、infra、deployment、dependency 变更，除非有审批证据。
- 要求 Provider 返回结构化报告。
- 捕获 changed files、diff summary、tests run、assumptions 和 risks。
- medium 及以上风险结果必须进入 Codex Review Gate。

## Adapter 输出证据

每次提交、状态查询和结果收集都应保留：

- provider name
- execution id
- attempt id
- external execution id
- status
- timestamps
- command 或 API operation summary
- changed files
- tests/checks run
- raw error summary
- failure classification rationale

证据必须足够让 Review Gate 复核，也必须避免泄漏 secrets。

## 边界规则

```text
Hermes controls state.
Provider adapter controls transport.
Provider performs work.
Review Gate decides quality.
Memory Center stores reusable lessons.
```
