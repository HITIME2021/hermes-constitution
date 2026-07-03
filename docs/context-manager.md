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

## Bilingual Context Policy

When both English and Chinese constitution files are available, Context Manager
may use Chinese files to clarify operator intent, but it must treat English
files as the protocol source of truth.

Provider context packets must not include duplicate bilingual rules. Context
Manager should normalize duplicated English/Chinese guidance into one rule and
preserve source references only when they help review or audit.

If bilingual sources disagree on a rule that affects execution, risk, provider
routing, review, memory, or approval, Context Manager must mark the conflict and
route it to Codex or the human operator instead of silently choosing both.

## Human-Facing Language Policy

For this operator environment, human-facing output should default to Simplified
Chinese (`zh-CN`).

This applies to:

- task summaries
- planning explanations
- review conclusions
- risk explanations
- memory candidates
- memory writeback text
- operator questions and approval prompts

Context packets must preserve protocol and implementation literals exactly:

- JSON/YAML field names
- schema identifiers
- file paths
- shell commands
- code symbols
- package names
- provider names
- quoted source text

Provider-facing packets may include English protocol terms when required by a
tool or schema, but their human-readable explanation should remain Chinese when
the operator is the audience.

## Constitution Snapshot Context

Normal sessions should use a pinned constitution snapshot instead of rereading
the full constitution repository every turn.

Default snapshot path:

```text
~/hermes-snapshots/current.md
```

Context Manager should load the full constitution only when the snapshot is
missing, stale, explicitly reloaded by the user, or a schema/policy conflict is
detected. See [Constitution Snapshot Policy](constitution-snapshot.md).
