# Workflow State Machine

Hermes v0.1 uses an engineering-oriented state machine.

## Main States

```text
created
clarifying
planned
routed
executing
reviewing
testing
completed
```

## Exceptional States

```text
blocked
replanning
failed
```

## State Responsibilities

### created

Hermes received a user goal and created a task id.

### clarifying

The task needs more information from the user.

### planned

Codex produced a task plan, risk estimate, and acceptance criteria.

### routed

Capability Resolver selected role, skill, provider, and review level.

### executing

Provider is executing the assigned task. For scoped implementation tasks this is
usually CodeBuddy.

Provider transport retries do not leave the task-level `executing` state. They
are tracked inside `ExecutionAttempt`.

### reviewing

Review Gate inspects the result.

### testing

Hermes runs or verifies required checks.

### completed

Task is done and eligible for memory writeback.

### blocked

Hermes cannot proceed without missing information, permission, environment, or
human decision.

### replanning

The original plan failed or became invalid. Codex analyzes and creates a new
plan.

Transport failures such as CLI timeout, network errors, rate limits, or
temporary provider unavailability must not enter `replanning` by default.

### failed

Retry limit was exceeded, the goal is unreachable, or the user stopped the task.

## Control Rule

Only Workflow Engine or Task Planner can change the main task state. Providers
submit events and results, but do not directly mutate task state.

## Transport Failure Rule

Hermes must not confuse provider channel failure with task failure.

```text
executing -> executing:
  transport_error with retry budget remaining

executing -> blocked:
  auth_error, quota exhaustion, provider unavailable after retry budget

executing -> replanning:
  semantic execution failure, review failure, invalid assumptions, repeated
  implementation failure

executing -> reviewing:
  execution completed
```

CLI timeout is a provider transport failure. It is not evidence that the
architecture, workflow, or task plan is wrong.
