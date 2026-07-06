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

Dashboard 可见的正式任务必须显式绑定 board、task、run 上下文。报告应包含 `board`、`task_id`、可用时的 `run_id`，以及精确的 `show`、`watch`、`runs` 命令。正式 Kanban CLI 命令必须传入 `--board <slug>`，不得依赖 current board。没有 `run_id` 的任务可以算 Kanban task 可见，但不能算已验证的 gateway-managed run。

Run logs 必须能按 board、task、run、phase 追溯。Dashboard 可见 comments 应只包含简短且不含敏感信息的摘要；详细本地证据应通过路径引用，不应直接粘贴到 comments。除非用户明确批准，logs 不得写入 git、长期记忆或上传。

Live provider orchestration 的本地 evidence 不得只有 summary。Evidence files 必须保留足够的 audit-grade details，使操作者不用重跑任务也能复盘发生了什么。最小证据集包括 exact commands run、git status before/after、`git diff --stat`、allowed files 的 scoped diff 或 patch summary、verification command and output summary、provider planning summary、execution summary、adversarial review verdict/findings、stop-condition checklist、final report fields，以及原始日志因安全原因被 redacted/withheld 时的说明。
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

## Board / Task / Run 上下文

Dashboard 可见的正式任务必须避免隐式 board state。

每个正式任务报告都应包含：

```text
board: <board_slug>
task_id: <task_id>
run_id: <run_id | none>
dashboard_url: <dashboard kanban URL>
show_command: hermes kanban --board <board_slug> show <task_id>
watch_command: hermes kanban --board <board_slug> watch
runs_command: hermes kanban --board <board_slug> runs <task_id>
```

正式 CLI 示例：

```bash
hermes kanban --board ghtip create "..."
hermes kanban --board ghtip show <task_id>
hermes kanban --board ghtip watch
hermes kanban --board ghtip runs <task_id>
```

正式任务不得依赖 current board。default board 和具名 board 使用不同的底层存储，未限定 board 的命令可能导致操作者、CLI 和 Dashboard 看到不同任务集。

Gateway-managed run 验证必须有 run lifecycle。

最小证据：

```text
claimed
spawned
heartbeat
completed | blocked | failed
```

如果一个任务有 comments 和 completion 但没有 `run_id`，它仍然可以是有价值的 Kanban task，但不得报告为已验证的 gateway-managed run。

## 可追溯日志

Run observability 必须保留足够证据用于审计，同时不能把 Dashboard comments 变成原始日志堆。

日志应分离：

- dashboard-visible summaries
- local evidence files

Dashboard 可见 comments：

- 使用简短且不含敏感信息的摘要
- 必要时引用本地 evidence path
- 不粘贴完整 provider stdout/stderr
- 不粘贴长 pytest 日志
- 不粘贴 secrets、tokens、`.env` 值或 credential material

推荐本地证据路径：

```text
~/.hermes/kanban/logs/<board>/<task_id>/<run_id>/
```

推荐文件：

```text
codex-planning.summary.md
codebuddy-execution.summary.md
verification.log
codex-review.summary.md
final-report.json
```

对于 live provider orchestration 任务，这些文件必须包含 audit-grade details，不得只有最终 summary。如果使用单个 `evidence.txt` 代替上述拆分文件，它仍然必须包含同样的最小证据集：

```text
commands_run
git_status_before
git_status_after
git_diff_stat
scoped_diff_or_patch_summary
verification_command
verification_output_summary
provider_planning_summary
execution_summary
adversarial_review_verdict
adversarial_review_findings
stop_condition_checklist
final_report_fields
redaction_or_withheld_notes
```

Evidence completeness 是最终结果的一部分。任务可以在 execution 和 verification 上通过，但如果 evidence path 存在却只有 summary，仍应报告 `evidence_completeness: needs_improvement`。

每个 evidence file 都应能追溯到：

```text
board
task_id
run_id
phase
provider
created_at
```

Final report 应引用 evidence path，不展开完整日志：

```text
evidence:
  verification_log: ~/.hermes/kanban/logs/ghtip/t_xxx/2/verification.log
  review_summary: ~/.hermes/kanban/logs/ghtip/t_xxx/2/codex-review.summary.md
```

Raw provider logs 只有在不含敏感信息或已经 redacted 时才可保存。如果 Hermes 无法判断日志是否包含 secrets，必须从 Dashboard 和 Memory 中 withheld 原始日志，并报告 `redacted` 或 `withheld` 及简短原因。

除非操作者明确批准，logs 不得提交到 git，不得写入长期 memory，不得上传到外部服务。

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
