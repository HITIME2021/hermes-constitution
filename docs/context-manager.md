# Context Manager

Context Manager turns full project reality into the minimum sufficient context
needed by a provider for the current task.

```text
Codex sees global reasoning context.
CodeBuddy sees scoped execution context.
```

## Responsibilities

- collect task, project, file, policy, skill, and memory context
- select relevant context
- compress long context
- build provider-specific context packets
- prevent providers from seeing unauthorized information
- record context feedback for future learning

## Context Scopes

```text
none
file
file_set
module
repo_map
full_project
```

Default mapping:

```text
trivial -> file or file_set
low -> file_set
medium -> file_set or module
high -> module + repo_map
critical -> repo_map + targeted files + Codex reasoning context
```

## Planning Context Packet

Used by Codex.

```yaml
planning_context_packet:
  task:
    user_goal: "Add login validation"
    constraints:
      - "Do not change auth API"
    acceptance_criteria:
      - "Invalid email should show error"
  project:
    repo_map: "summarized architecture"
    tech_stack:
      - react
      - typescript
    conventions:
      - "Use existing validation helpers"
  relevant_files:
    - path: src/components/LoginForm.tsx
      reason: primary target
  memory_hints:
    - "Project prefers no new validation dependencies."
  risk:
    preliminary_level: medium
```

## Execution Context Packet

Used by CodeBuddy.

```yaml
execution_context_packet:
  task_id: task-001
  role: frontend-developer
  skill: react-component-builder
  goal: "Add email validation to LoginForm before submit."
  allowed_files:
    - src/components/LoginForm.tsx
    - src/components/LoginForm.test.tsx
  forbidden_files:
    - src/auth/session.ts
    - src/api/auth.ts
    - db/**
  constraints:
    - "Do not change public API."
    - "Do not add dependencies."
    - "Keep changes minimal."
  acceptance_criteria:
    - "Invalid email displays a clear error."
    - "Valid login flow behaves as before."
  report_contract:
    required:
      - files_changed
      - summary
      - tests_run
      - assumptions
      - risks
```

## Non-Compressible Fields

These fields must never be removed by context compression:

- acceptance criteria
- allowed scope
- forbidden scope
- risk controls
- approval requirements
- report contract

## Security Rules

- Secrets, tokens, and credentials never enter provider context.
- CodeBuddy does not receive full Hermes internal reasoning.
- Forbidden files may be named by path but not provided as content.
- User memory is only shared when directly relevant and appropriately filtered.

