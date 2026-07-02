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

## Failure Types

```text
unclear_requirement
missing_context
permission_denied
scope_conflict
test_failure
implementation_failed
tool_failure
policy_violation
unexpected_repo_state
```

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

## Provider Adapter

Provider adapters translate Hermes protocol into provider-native execution.

```text
submit_execution(request) -> execution_id
get_status(execution_id) -> provider_status
collect_result(execution_id) -> execution_result
cancel(execution_id) -> cancel_result
```

## Hard Rules

- Provider only executes within `allowed_scope`.
- Provider cannot expand scope.
- Provider cannot lower risk.
- Provider must return structured results.
- Failures must include type and evidence.
- Permission gaps require `permission_request`.
- Code changes must capture diff.
- CodeBuddy result must pass Review Gate before delivery.

