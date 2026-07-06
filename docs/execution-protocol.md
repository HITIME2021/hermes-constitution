# Execution Protocol

Execution Protocol defines how Hermes assigns work to providers and how
providers report back.

## Core Rule

```text
Hermes controls state.
Provider performs work.
Review Gate decides quality.
```

Providers cannot directly change task state.

## ExecutionRequest

```yaml
execution_request:
  id: exec-001
  task_id: task-001
  created_by: hermes
  assigned_provider: codebuddy
  execution_mode: live
  mode: stable
  risk_level: medium
  role:
    id: frontend-developer
    division: engineering
  skill:
    id: react-component-builder
    version: 0.1.0
  goal: "Add email validation to LoginForm before submit."
  allowed_scope:
    files:
      - src/components/LoginForm.tsx
      - src/components/LoginForm.test.tsx
    commands:
      - "npm test -- LoginForm"
  forbidden_scope:
    files:
      - src/auth/session.ts
      - src/api/auth.ts
      - db/**
    actions:
      - add_dependency
      - change_public_api
      - delete_files
      - access_secrets
  acceptance_criteria:
    - "Invalid email displays an error before submit."
    - "Valid login flow remains unchanged."
  report_contract:
    required_fields:
      - status
      - files_changed
      - summary
      - tests_run
      - assumptions
      - risks
      - validation_result
```

## ExecutionResult

```yaml
execution_result:
  id: result-001
  execution_id: exec-001
  task_id: task-001
  provider: codebuddy
  status: completed
  files_changed:
    - src/components/LoginForm.tsx
  summary:
    - "Added email validation before submit."
  tests_run:
    - command: "npm test -- LoginForm"
      status: passed
      output_summary: "5 tests passed"
  assumptions:
    - "Existing submit handler should remain unchanged."
  risks:
    - "No browser manual validation performed."
  validation_result:
    passed: true
```

## Dry-Run Execution

Dry-run mode simulates planning, routing, scope construction, review planning,
and report contracts without calling a real provider or mutating files.

Dry-run requests must still name the intended provider in `assigned_provider`.
Hermes must not invent a pseudo-provider such as `agent` to represent dry-run
execution. Use `execution_mode: dry_run` instead.

```yaml
execution_request:
  id: exec-dryrun-001
  task_id: task-dryrun-001
  created_by: hermes
  assigned_provider: codebuddy
  execution_mode: dry_run
  mode: stable
  risk_level: trivial
```

In dry-run mode:

- Provider adapters are not invoked.
- Files are not modified.
- Memory writeback is disabled unless explicitly approved.
- The output must remain schema-valid.
- The ReviewPlan should describe what would be checked after live execution.

## Failure Types

```text
unclear_requirement
missing_context
permission_denied
needs_user_confirmation
scope_conflict
test_failure
implementation_failed
tool_failure
policy_violation
unexpected_repo_state
```

## Failure Categories

Provider failures must be classified before they reach Workflow Engine.

Hermes must distinguish transport failures from task failures.

```text
transport_error
  The provider channel failed. Examples: CLI timeout, network error, rate limit,
  authentication error, provider unavailable, CLI crash, malformed response.

provider_error
  The provider process ran but could not complete its assigned attempt for
  provider-local reasons. Examples: tool unavailable, sandbox setup failed,
  adapter protocol mismatch.

execution_error
  The task attempt semantically failed. Examples: tests failed, implementation
  incomplete, review failed, scope conflict, missing context, invalid assumption.
```

Transport failure is not evidence that the task plan, architecture, or workflow
is wrong.

## ExecutionAttempt

Transport retries are tracked as execution attempts, not as task replanning.

```yaml
execution_attempt:
  id: attempt-001
  execution_id: exec-001
  provider: codex
  status: transport_failed
  started_at: "2026-07-03T00:00:00Z"
  ended_at: "2026-07-03T00:02:00Z"
  error:
    category: transport_error
    type: timeout
    retryable: true
    counts_as_task_failure: false
    should_trigger_replanning: false
```

Attempt statuses:

```text
pending
running
succeeded
transport_failed
provider_failed
execution_failed
timed_out
cancelled
```

## Transport Error Policy

```text
timeout:
  Retry the provider call with bounded backoff. Do not replan the task.

network_error:
  Retry with bounded backoff. Do not replan the task.

rate_limit:
  Back off, delay, or route to an allowed fallback provider. Do not replan the
  task unless the fallback changes task semantics.

auth_error:
  Move to blocked and wait for configuration repair.

needs_user_confirmation:
  Stop and ask the operator unless the prompt is only the provider's
  non-interactive scoped execution confirmation for an already approved exact
  ExecutionRequest. Do not use `-y`/`--yes` for auth, dependency install,
  network, destructive, elevated, provider settings, git commit/push,
  credential, or scope-expanding actions.

provider_unavailable:
  Retry or mark blocked depending on retry budget.

cli_crash:
  Restart adapter or mark blocked. Do not assume task failure.

malformed_response:
  Retry once if safe. Repeated malformed responses indicate adapter/provider
  issue, not task architecture failure.
```

Only semantic execution failures, review failures, repeated implementation
failures, invalid assumptions, or newly discovered constraints may trigger
`replanning`.

## Human Intervention Budget

Execution retries and semantic revisions are bounded. Hermes must stop and ask
for human intervention when repeated failure suggests missing context, wrong
direction, unsafe scope, or wasteful token burn.

Default v0.1 budgets:

```yaml
human_intervention_defaults:
  transport_retry_budget: 2
  semantic_revision_budget: 2
  replanning_budget: 1
  malformed_output_budget: 1
  clarification_budget: 1
  provider_switch_budget: 1
```

When a budget is exhausted, Hermes emits `human_intervention_request` and moves
the task to `blocked` instead of continuing automatic execution.

See [Human Intervention Policy](human-intervention-policy.md).

## PermissionRequest

```yaml
permission_request:
  execution_id: exec-001
  provider: codebuddy
  requested_action: modify_file
  requested_scope:
    files:
      - src/validation/auth.ts
  reason: "Existing validation helper needs an export adjustment."
  risk_impact:
    current: medium
    proposed: medium
```

## Codex Scoped Repair

Codex may be assigned execution work when review determines that a CodeBuddy
revision is likely insufficient or has already failed. This is an execution
phase, not a review phase.

Required boundaries:

- create a new ExecutionRequest
- set `assigned_provider: codex`
- use a narrow `allowed_scope`
- preserve the Review Gate level required by risk
- record why CodeBuddy revision was not enough
- do not let the reviewer approve its own unreviewed repair
- request human approval when scope, dependency, auth, database, security,
  infrastructure, deployment, or policy risk requires it

`codex_scoped_repair` should be used sparingly. It is a precision escalation
path for complex findings, not a replacement for CodeBuddy as the default
scoped executor.

## Provider Adapter

Provider adapters translate Hermes protocol into provider-native execution.

```text
submit_execution(request) -> execution_id
get_status(execution_id) -> provider_status
collect_result(execution_id) -> execution_result
cancel(execution_id) -> cancel_result
```

Provider adapters must return classified errors.

```yaml
provider_error:
  category: transport_error
  type: timeout
  provider: codex
  retryable: true
  counts_as_task_failure: false
  counts_as_provider_reliability_signal: true
  should_trigger_replanning: false
  evidence:
    - "Codex CLI timed out after 120 seconds."
```

## Hard Rules

- Provider only executes within `allowed_scope`.
- Dry-run must use `execution_mode: dry_run`; it must not create fake providers.
- Provider cannot expand scope.
- Provider cannot lower risk.
- Provider must return structured results.
- Failures must include type and evidence.
- Transport failures must not trigger task replanning by default.
- Timeout is not evidence that the architecture, workflow, or task plan is wrong.
- Provider adapters must classify failures before returning them to Workflow Engine.
- Auth errors and quota/rate-limit errors move the task to blocked or delayed, not failed.
- Transport retries are bounded and recorded as ExecutionAttempt, not Task retry.
- Permission gaps require `permission_request`.
- Provider interactive confirmations require operator approval unless they are
  limited to pre-approved non-interactive scoped execution inside the exact
  ExecutionRequest.
- Code changes must capture diff.
- CodeBuddy result must pass Review Gate before delivery.
- Repeated semantic failure, repeated review failure, or exhausted retry budget requires human intervention.
