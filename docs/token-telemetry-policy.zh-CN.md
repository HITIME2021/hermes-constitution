# Token Telemetry Policy

Token telemetry 记录 provider 和 tool 的上下文使用情况，用于 audit、成本感知、
prompt 优化和 planning mode 评估。

Token telemetry 可以帮助 Hermes 判断一次 run 是否高效，但不能单独判断一次 run
是否高质量。

## 核心规则

<!-- snapshot:block id="token-telemetry-policy.zh-CN" section="Token Telemetry" priority="87" -->
除非由 provider 直接报告，否则 token telemetry 是 audit evidence，不是 billing
truth。Hermes 不得伪造 token counts。

每个 provider 或 tool phase 都应在可获得时记录 token telemetry。如果 provider 或
tool 不暴露 token usage，Hermes 必须把字段记录为 `unavailable`。如果 Hermes
估算 usage，source 必须标记为 `estimated`。

Token telemetry 必须与 quality telemetry 配对。Hermes 不得只优化低 token 使用。
Planning-mode comparison、provider selection、prompt distillation 和 frontend
artifact assistance 都必须同时考虑 cost 和 quality：

```text
token usage + task quality + review verdict + regression result + evidence completeness
```

当 telemetry 存在时，task final report 应包含精简 token/quality table；如果 token
data 不可获得，也必须明确说明。

Telemetry source values：

```text
cli_reported
api_reported
estimated
unavailable
```

规则：

- 只有 CLI 直接报告 token usage 时，才能使用 `cli_reported`
- 只有 API response 直接报告 token usage 时，才能使用 `api_reported`
- 只有 Hermes 明确估算 usage 时，才能使用 `estimated`
- 无法获得 usage 时使用 `unavailable`
- 不得把 estimate 伪装成 provider-reported usage
- 不得从 latency、成本感觉或 model name 推断精确 token counts
- 不得记录 raw secrets、credentials、tokens、cookies 或完整 private prompts
- 必须保留足够 prompt summary 和 evidence references 以支持 audit
- final report 必须包含 quality outcome，不能只报告 token usage
<!-- /snapshot:block -->

## Telemetry File

推荐路径：

```text
~/.hermes/kanban/logs/<board>/<task_id>/<run_id>/token-telemetry.json
```

Telemetry file 应从 `evidence.txt` 和 final report 中引用。

## Schema

```yaml
token_telemetry:
  task_id: t_example
  run_id: run-001
  constitution_version: abc1234
  planning_source_of_record: codex_native
  total:
    prompt_tokens: null
    completion_tokens: null
    total_tokens: null
    estimated_cost: null
    source: unavailable
  provider_chain:
    - phase: codex_planning
      provider: codex
      tool_classification: backend_tool
      model: gpt-5.5
      prompt_tokens: null
      completion_tokens: null
      total_tokens: null
      estimated_cost: null
      source: unavailable
      evidence_ref: evidence.txt#codex_planning
    - phase: codebuddy_execution
      provider: codebuddy
      tool_classification: backend_tool
      model: deepseek-v4-pro
      prompt_tokens: null
      completion_tokens: null
      total_tokens: null
      estimated_cost: null
      source: unavailable
      evidence_ref: evidence.txt#codebuddy_execution
  prompt_distillation:
    original_context_estimate: null
    distilled_context_estimate: null
    compression_ratio: null
    omitted_context_reason: null
    preserved_evidence:
      - allowed_scope
      - forbidden_scope
      - stop_conditions
  quality:
    final_status: pass
    review_verdict: approved_with_notes
    tests_result: "247/247 passed"
    regression_result: none_detected
    evidence_completeness: audit_grade
    stop_conditions_triggered: []
```

不可获得的数值使用 `null`。不要用 `0`，除非 provider 真实报告为 zero。

## Final Report Table

可获得时，final report 应包含精简表格：

```text
phase | provider | model | tokens | source | quality signal
```

示例：

```text
codex_planning      | codex    | gpt-5.5        | unavailable | unavailable | plan accepted
codebuddy_execution | codebuddy | deepseek-v4-pro | unavailable | unavailable | scoped diff valid
codex_review        | codex    | gpt-5.5        | unavailable | unavailable | approved_with_notes
```

如果数据不可获得，报告应明确写出，而不是省略该行。

## Quality Signals

Token data 必须结合 quality signals 解释：

- final status
- review verdict
- tests run and result
- regression status
- stop conditions triggered
- bounded revisions 次数
- evidence completeness
- operator intervention count
- planning mode 是否合适

如果一次 run 的 evidence 弱、review 质量差、测试缺失、隐藏扩大 scope，或仍有未解决
defects，那么更低 token usage 不是成功。

## Planning Mode Evaluation

比较 `codex_native` 和 `frontend_artifact_assisted` 时，Hermes 应比较：

```text
planning tokens
review tokens
execution retries
clarification rounds
review findings
test/regression outcome
evidence completeness
operator intervention count
time to approved ExecutionRequest
```

只有当 `frontend_artifact_assisted` 在不降低 task quality、review quality 和
auditability 的前提下降低整体成本或减少歧义时，才应视为有收益。

## Privacy and Redaction

Telemetry 不得包含：

- raw credentials
- API keys
- auth tokens
- cookies
- `.env` values
- 有 summary 即足够时的完整 private prompts
- provider home credential/config file contents

如果 prompt content 被 summary 替代，应记录：

```yaml
redaction:
  raw_prompt_recorded: false
  reason: private_context_or_unnecessary_for_audit
  summary_ref: evidence.txt#prompt_summary
```

## 非目标

Token telemetry 不是 billing system。

Token telemetry 不是跳过 Codex review、approval gates、stop conditions 或 evidence
的理由。

当任务风险需要更强 planning 或 review 时，Token telemetry 不得推动 Hermes 选择便宜但
低质量的执行方式。
