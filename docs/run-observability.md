# Run Observability

Run observability defines how Hermes exposes live task progress to the operator
without turning the dashboard into the execution authority.

## Position

Hermes Dashboard and Kanban are observability and coordination surfaces.

They may show task state, run attempts, events, comments, summaries, assignees,
and stop-condition status. They do not replace Project Policy, Provider
Adapter boundaries, Review Gate, or explicit operator approval.

## Provider Orchestration Visibility

<!-- snapshot:block id="provider-orchestration-observability" section="Run Observability" priority="60" -->
When dashboard visibility is required, provider orchestration live tests should
run as Hermes-managed Kanban tasks or write equivalent Kanban events. External
CLI calls to Codex or CodeBuddy are not automatically visible in Dashboard
unless Hermes records them as task/run events.

Dashboard is an observability surface, not execution authority. Kanban events,
comments, heartbeats, and run summaries may record provider orchestration
phases such as Codex planning, CodeBuddy execution, verification, and Codex
review. These records must obey existing Provider Adapter, auth, dependency,
memory, and human-intervention policies.

Provider orchestration comments should use the stable marker
`[provider-orchestration]` with at least `phase`, `provider`, `status`, and
`summary`. Optional fields may include `run_id`, `evidence`, `files_changed`,
`tests_run`, `stop_condition`, and `next_action`.

Dashboard-visible formal tasks must bind board, task, and run context
explicitly. Reports should include `board`, `task_id`, `run_id` when available,
and exact `show`, `watch`, and `runs` commands. Formal Kanban CLI commands must
pass `--board <slug>` instead of relying on the current board. A task without a
`run_id` is observable as a Kanban task, but it is not validated as a
gateway-managed run.
<!-- /snapshot:block -->

Expected phase events:

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

Events may be represented as Kanban comments, task events, run metadata,
heartbeats, or structured gateway events. The exact transport can evolve, but
the operator must be able to inspect the current phase and final result without
reading raw provider logs first.

## Provider Orchestration Comment Convention

Provider orchestration comments should use a stable marker so humans, Hermes,
and future tooling can identify phase records without requiring a new event
system.

Minimum format:

```text
[provider-orchestration]
phase: <phase_name>
provider: <provider_role>
status: <started|completed|passed|approved|blocked|failed>
summary: <short non-sensitive summary>
```

Required fields:

- `phase`
- `provider`
- `status`
- `summary`

Optional fields:

- `run_id`
- `evidence`
- `files_changed`
- `tests_run`
- `stop_condition`
- `next_action`

Example:

```text
[provider-orchestration]
phase: verification.passed
provider: local
status: passed
summary: pytest 6/6 passed.
tests_run: pytest tests/
evidence: tests/test_normalize_label.py
```

Blocked example:

```text
[provider-orchestration]
phase: codex_review.blocked
provider: codex
status: blocked
stop_condition: review_scope_violation
summary: Codex review found a modification outside allowed_scope.
next_action: ask operator for scope approval or revert the out-of-scope diff.
```

This convention does not replace native Kanban events. It is a lightweight
comment contract layered on top of existing Kanban comments/events.

## Board / Task / Run Context

Dashboard-visible formal tasks must avoid implicit board state.

Every formal task report should include:

```text
board: <board_slug>
task_id: <task_id>
run_id: <run_id | none>
dashboard_url: <dashboard kanban URL>
show_command: hermes kanban --board <board_slug> show <task_id>
watch_command: hermes kanban --board <board_slug> watch
runs_command: hermes kanban --board <board_slug> runs <task_id>
```

Formal CLI examples:

```bash
hermes kanban --board ghtip create "..."
hermes kanban --board ghtip show <task_id>
hermes kanban --board ghtip watch
hermes kanban --board ghtip runs <task_id>
```

Do not rely on the current board for formal tasks. The default board and named
boards use different backing stores, so an unqualified command can make the
operator, CLI, and Dashboard inspect different task sets.

Gateway-managed run validation requires a run lifecycle.

Minimum evidence:

```text
claimed
spawned
heartbeat
completed | blocked | failed
```

If a task has comments and completion but no `run_id`, it may still be a useful
Kanban task, but it must not be reported as a validated gateway-managed run.

## Safety Rules

Run observability must not leak secrets or credential material.

Forbidden in dashboard-visible logs or events:

- API keys, tokens, cookies, OAuth refresh tokens
- `.env` values
- provider credential file contents
- full private prompts when they contain secrets
- raw provider stdout/stderr that includes sensitive data

Allowed:

- phase name
- provider role
- provider CLI version
- status
- changed file summary
- test command summary
- test result
- review verdict
- stop-condition status
- short non-sensitive error summary

## Gateway and Dashboard Boundary

Gateway may execute or dispatch Hermes-managed tasks.

Dashboard may display, filter, and inspect tasks, runs, and events. It must not
silently bypass approvals or directly mutate provider execution state outside
declared Kanban or command-handler operations.

If dashboard or gateway is unavailable, Hermes may still execute through CLI,
but the final report must state that live dashboard visibility was unavailable.

## Initial Test Scope

The first provider orchestration observability test should use an isolated
sample project, not a production project.

Recommended scope:

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

The test must not modify:

```text
~/.hermes/hermes-agent
~/projects/hermes-constitution
~/projects/GitHub-Trend-Intelligence-Platform
provider credential directories
```

The test should validate that Dashboard can observe:

- task created
- task assigned
- run claimed
- provider planning started/completed
- provider execution started/completed
- verification result
- review result
- final task completion or block
