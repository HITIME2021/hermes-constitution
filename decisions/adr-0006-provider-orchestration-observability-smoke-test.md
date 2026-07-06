# ADR 0006: Provider Orchestration Observability Smoke Test

## Status

Accepted.

## Date

2026-07-06.

## Context

Hermes v0.2 defines Dashboard and Kanban as observability surfaces for
Hermes-managed provider orchestration runs.

The previous provider orchestration tests verified that Codex planning,
CodeBuddy scoped execution, pytest verification, and Codex review could run.
This test verified that the same kind of provider chain can be observed through
a Hermes-managed Kanban task and run lifecycle.

## Test

Task:

```text
v0.2 observability smoke: provider orchestration visible in dashboard
```

Identifiers:

```text
task_id: t_cb06d915
run_id: 2
assignee: default
```

Target project:

```text
~/projects/hermes-kanban-provider-smoke
```

Provider chain:

```text
Codex planning
  -> CodeBuddy execution
  -> pytest verification
  -> Codex review
```

Implementation task:

```text
Create normalize_label(value: str) -> str.
```

Expected behavior:

- trim surrounding whitespace
- collapse consecutive internal whitespace to one space
- return an empty string for empty or whitespace-only input

## Result

The v0.2 provider orchestration observability smoke test passed.

Changed files:

```text
normalize_label.py
tests/test_normalize_label.py
```

Verification:

```text
pytest: 6/6 passed
Codex review: approved
human_intervention_stop_conditions: none triggered
final_status: pass
```

Dashboard / Kanban observability:

```text
created -> claimed -> spawned -> heartbeat -> comments -> completed
```

The run completed successfully:

```text
run_id: 2
outcome: completed
elapsed: 175s
```

All required stage events were recorded as Kanban comments:

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

The completion summary stated that the Codex -> CodeBuddy -> Codex pipeline
completed successfully and that all stage events were observable through
Dashboard/Kanban.

## Decision

Hermes v0.2 accepts Kanban-managed provider orchestration observability as
validated for isolated smoke-test tasks.

This confirms:

- external provider orchestration should be wrapped as a Hermes-managed Kanban
  task when live Dashboard visibility is required
- Kanban comments/events are sufficient for v0.2 phase visibility
- run history can preserve provider orchestration attempts
- Dashboard is useful as an observability surface but remains outside execution
  authority

## Consequences

Positive:

- Provider orchestration can now be observed through the built-in Hermes
  Dashboard/Kanban system.
- A separate dashboard project is not needed for v0.2.
- Future live tests can require Kanban comments or events for every major
  orchestration phase.

Remaining limits:

- This was an isolated sample project, not a production-project run.
- The current phase records are comments, not a dedicated typed event schema.
- More complex tasks may need structured event payloads, log links, and
  stop-condition fields.
- Secrets and raw provider logs must remain excluded from dashboard-visible
  comments and summaries.
