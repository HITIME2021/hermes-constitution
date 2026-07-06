# Run Observability

Run Observability 定义 Hermes 如何把 live task 进度暴露给操作者，同时不把 Dashboard 变成执行权威。

## 定位

Hermes Dashboard 和 Kanban 是可观测性与协作界面。

它们可以展示 task state、run attempts、events、comments、summaries、assignees 和 stop-condition status。它们不能替代 Project Policy、Provider Adapter 边界、Review Gate 或明确的 operator approval。

## Provider Orchestration 可视化

<!-- snapshot:block id="provider-orchestration-observability.zh-CN" section="Run Observability" priority="61" -->
当需要 Dashboard 可视化时，provider orchestration live test 应作为 Hermes-managed Kanban task 运行，或写入等价的 Kanban events。外部终端中直接调用 Codex 或 CodeBuddy 的 CLI 不会自动出现在 Dashboard 中，除非 Hermes 把这些阶段记录为 task/run events。

Dashboard 是可观测性界面，不是执行权威。Kanban events、comments、heartbeats 和 run summaries 可以记录 Codex planning、CodeBuddy execution、verification、Codex review 等 provider orchestration 阶段。这些记录必须继续遵守 Provider Adapter、auth、dependency、memory 和 human-intervention 策略。

Provider orchestration comments 应使用稳定 marker `[provider-orchestration]`，并至少包含 `phase`、`provider`、`status`、`summary`。可选字段包括 `run_id`、`evidence`、`files_changed`、`tests_run`、`stop_condition`、`next_action`。
<!-- /snapshot:block -->

建议记录的阶段事件：

```text
codex_planning.started
codex_planning.completed
codebuddy_execution.started
codebuddy_execution.completed
verification.started
verification.passed
codex_review.started
codex_review.approved
final_report.completed
```

事件可以表现为 Kanban comment、task event、run metadata、heartbeat 或结构化 gateway event。具体传输方式可以演进，但操作者必须能在不先阅读原始 provider 日志的情况下看到当前阶段和最终结果。

## Provider Orchestration Comment 约定

Provider orchestration comments 应使用稳定 marker，方便人类、Hermes 和未来工具识别阶段记录，而不需要新建一套 event system。

最小格式：

```text
[provider-orchestration]
phase: <phase_name>
provider: <provider_role>
status: <started|completed|passed|approved|blocked|failed>
summary: <short non-sensitive summary>
```

必填字段：

- `phase`
- `provider`
- `status`
- `summary`

可选字段：

- `run_id`
- `evidence`
- `files_changed`
- `tests_run`
- `stop_condition`
- `next_action`

示例：

```text
[provider-orchestration]
phase: verification.passed
provider: local
status: passed
summary: pytest 6/6 passed.
tests_run: pytest tests/
evidence: tests/test_normalize_label.py
```

阻塞示例：

```text
[provider-orchestration]
phase: codex_review.blocked
provider: codex
status: blocked
stop_condition: review_scope_violation
summary: Codex review found a modification outside allowed_scope.
next_action: ask operator for scope approval or revert the out-of-scope diff.
```

该约定不替代 Kanban 原生 events。它是在现有 Kanban comments/events 之上的轻量 comment contract。

## 安全规则

Run observability 不得泄露 secrets 或 credential material。

Dashboard 可见日志或事件中禁止出现：

- API keys、tokens、cookies、OAuth refresh tokens
- `.env` 值
- provider credential 文件内容
- 包含 secrets 的完整私有 prompt
- 包含敏感信息的原始 provider stdout/stderr

允许出现：

- phase name
- provider role
- provider CLI version
- status
- changed file summary
- test command summary
- test result
- review verdict
- stop-condition status
- 简短且不含敏感信息的 error summary

## Gateway 与 Dashboard 边界

Gateway 可以执行或调度 Hermes-managed tasks。

Dashboard 可以展示、过滤和检查 tasks、runs 与 events。它不得静默绕过 approvals，也不得在声明的 Kanban 或 command-handler 操作之外直接修改 provider execution state。

如果 dashboard 或 gateway 不可用，Hermes 仍可通过 CLI 执行，但 final report 必须说明 live dashboard visibility 不可用。

## 初始测试范围

第一次 provider orchestration observability test 应使用隔离样例项目，不使用生产项目。

推荐范围：

```text
target_project: ~/projects/hermes-kanban-provider-smoke
risk_level: medium
review_level: L2
execution_mode: live
providers:
  planning: codex
  execution: codebuddy
  review: codex
```

测试不得修改：

```text
~/.hermes/hermes-agent
~/projects/hermes-constitution
~/projects/GitHub-Trend-Intelligence-Platform
provider credential directories
```

测试应验证 Dashboard 能观察到：

- task created
- task assigned
- run claimed
- provider planning started/completed
- provider execution started/completed
- verification result
- review result
- final task completion or block
