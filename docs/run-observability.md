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
