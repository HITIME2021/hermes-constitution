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

### failed

Retry limit was exceeded, the goal is unreachable, or the user stopped the task.

## Control Rule

Only Workflow Engine or Task Planner can change the main task state. Providers
submit events and results, but do not directly mutate task state.

