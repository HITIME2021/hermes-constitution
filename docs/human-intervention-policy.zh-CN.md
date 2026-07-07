# 人工介入策略

Hermes 是自动化系统，但不能无限循环。人工介入是控制时间、token、项目安全和方向正确性的边界。

本策略定义 Hermes 什么时候必须停止自动重试或重规划，并请求操作者决策。

## 核心规则

```text
自动化可以重试有边界的运行故障。
自动化可以修正有边界的语义失败。
当重复失败说明缺少判断、缺少上下文、方向错误、范围不安全或 token 消耗不值得时，必须停止。
```

触发停止条件后，Hermes 将任务转为 `blocked`，并输出 Human Intervention Request。

## Human Intervention Request

```yaml
human_intervention_request:
  task_id: task-001
  reason: repeated_revision_failure
  current_state: executing
  recommended_next_state: blocked
  summary: "CodeBuddy 两次修正仍未满足同一个验收标准。"
  attempts:
    execution_attempts: 2
    revision_cycles: 2
    replans: 1
  evidence:
    - "Review 两次指出同一个 validation 行为缺失。"
    - "测试仍通过，但验收标准未满足。"
  options:
    - approve_one_more_attempt
    - change_scope
    - ask_codex_to_replan
    - stop_task
  default_recommendation: ask_codex_to_replan
```

## 停止条件

出现以下任一情况，Hermes 必须停止并请求人工介入。

| 条件 | 阈值 | 必须动作 |
|------|------|----------|
| 同一语义失败重复 | 2 次 revision cycle 失败 | 阻塞并询问用户 |
| 同一测试失败重复 | 2 次 attempt 出现同一测试失败 | 阻塞并询问用户 |
| 同一 review finding 重复 | 2 次 review 出现同一 required action | 阻塞并询问用户 |
| 重规划后仍无进展 | 已经 replan 1 次 | 再次 replan 前必须询问 |
| Provider 传输失败耗尽 retry budget | retry budget 用尽 | 阻塞，默认不重规划 |
| Provider malformed output 重复 | 2 次 malformed response | 作为 provider/adapter 问题阻塞 |
| 需要扩大范围 | 任意 forbidden/protected scope 请求 | 请求用户或批准人 |
| 依赖清单或 lockfile 变更 | 命中 dependency profile | 请求用户 |
| 任务前提在目标 checkout 中不存在 | 任意引用的文件、符号或行为缺失 | Provider 执行前阻塞 |
| secrets/auth/db/infra/deployment 写入或变更 | 任意一次 | 请求用户；critical 需要 L4 |
| 验收标准冲突 | 任意冲突 | 请求用户 |
| 澄清后仍不明确 | 1 次 clarification cycle 后仍不清楚 | 请求用户 |
| token/time 成本不值得 | 预计额外工作超过任务价值 | 请求用户 |
| 生成计划违反宪法 | 任意违反 | 停止并请求用户 |

<!-- snapshot:block id="dependency-human-intervention.zh-CN" section="Human Intervention" priority="51" -->
只要命中目标项目的 dependency profile，dependency manifest 或 lockfile 变更就必须触发人工介入。即使具体 manifest 或 lockfile 文件名因生态不同而变化，也必须按 dependency change 处理。
<!-- /snapshot:block -->
## 任务前提校验

Live execution 前，Hermes 必须在当前选定的 target worktree / commit 中验证任务前提是否成立。

任务前提包括：引用的文件、符号、函数、命令、测试、配置键、文档行为和 bug 描述。它们必须来自当前目标 checkout，不能从 dirty working directory、旧 smoke worktree、无关分支、聊天历史或 memory 推断。

如果任务引用的 symbol、file 或 behavior 在目标 worktree 中不存在，Hermes 必须在调用 CodeBuddy 前停止：

```yaml
stop_condition: task_premise_invalid_target_absent
status: blocked_pending_operator_approval
provider_execution_started: false
codebuddy_called: false
```

Evidence 必须记录 target_project、worktree_path、git_branch、git_commit、git_status、premise_checked、lookup_commands、lookup_result、source_of_false_premise（如已知）以及 files_changed: 0。

## 默认预算

除非 Project Policy 更严格，v0.1 默认如下：

```yaml
human_intervention_defaults:
  transport_retry_budget: 2
  semantic_revision_budget: 2
  replanning_budget: 1
  malformed_output_budget: 1
  clarification_budget: 1
  provider_switch_budget: 1
```

预算按 task 计算，不按 provider message 计算。如果 CodeBuddy、Codex 或未来 Provider 遇到的是同一个语义问题，必须累计计算。

## Retry 与人工介入的边界

以下有边界的运行故障可以自动重试：

- timeout
- network error
- rate limit with backoff
- temporary provider unavailable
- safe malformed response retry

以下判断型失败必须人工介入：

- 反复实现失败
- 反复 review rejection
- 需求不清或冲突
- 需要扩大 scope
- dependency manifest 或 lockfile 变更
- 风险高于原路由判断
- provider 输出看似合理但方向错误
- token 消耗相对任务价值已经不划算

## Token 浪费防护

Hermes 不得因为“技术上还能继续尝试”就继续自动循环。

第一次语义修正失败后，每次继续尝试前都必须检查：

- 失败是否和上次相同？
- 是否有新信息？
- 计划是否有实质变化？
- 剩余工作是否值得再次调用 provider？
- 人工判断是否比再次自动尝试更便宜？

如果答案不明确，停止并询问操作者。

## 状态机集成

```text
executing -> blocked:
  触发人工介入停止条件

reviewing -> blocked:
  重复 review finding 或需要批准

replanning -> blocked:
  replan budget 用尽或战略方向不清

blocked -> planned:
  用户修改 scope、验收标准或计划

blocked -> executing:
  用户批准再做一次有边界的尝试

blocked -> failed:
  用户停止任务或确认目标不可达
```

## 硬规则

- 停止条件高于 Provider 偏好。
- 停止条件高于 Skill confidence。
- 停止条件高于自动化惯性。
- CodeBuddy 不能在重复 review 失败后继续自动修正，除非 Codex 或用户决策。
- Codex 不能在 replan budget 之外反复重规划，除非用户批准。
- 人工批准必须明确，并作为 evidence 记录。
- dry-run 可以建议人工介入，但不能创建真实 approval state。
