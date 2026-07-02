# Memory Center

Memory Center saves, retrieves, and updates reusable experience. It is not a
raw log store and not a replacement for current task reasoning.

## Memory Types

```text
User Memory
Project Memory
Execution Memory
Provider Memory
Skill Memory
```

## User Memory

Stores stable user preferences.

Examples:

- user prefers stable mode by default
- user wants Codex to handle architecture and review
- user wants CodeBuddy to handle low-cost execution

## Project Memory

Stores stable project facts and conventions.

Examples:

- project uses React and TypeScript
- auth logic lives under `src/auth`
- new validation dependencies are discouraged

## Execution Memory

Stores reusable lessons from task execution.

Examples:

- similar LoginForm tasks should include validation helper files
- a specific test command quickly validates a module
- a missing context pattern caused a failed attempt

## Provider Memory

Stores provider performance history.

Examples:

- CodeBuddy performs well on scoped frontend changes
- CodeBuddy performs poorly on auth logic changes

## Skill Memory

Stores skill performance and project-specific refinements.

Examples:

- `react-component-builder` should include accessibility checks in this project
- dependency upgrade tasks need Codex planning first

## Write Pipeline

```text
Execution Result
  -> Review Gate
  -> Memory Candidate Extractor
  -> Codex Memory Synthesizer
  -> Memory Quality Gate
  -> Memory Center
```

CodeBuddy cannot directly write long-term memory.

## Memory Candidate

```yaml
memory_candidate:
  source_task_id: task-001
  type: execution_memory
  claim: "Similar LoginForm tasks should include src/validation/auth.ts."
  evidence:
    - "Initial execution missed the existing validation helper."
    - "Review identified duplicated validation logic."
  confidence: medium
  scope: project
```

## Quality Gate

Before writing memory, Hermes asks:

- is this reusable?
- is there evidence?
- is it only a one-time event?
- does it conflict with existing memory?
- can it become stale?
- does it contain sensitive information?
- does it require user confirmation?

## Hard Rules

- Memory must include claim, evidence, and confidence.
- Memory cannot contain secrets, tokens, or credentials.
- Memory cannot bypass Risk Evaluator.
- Memory can adjust scoring but cannot relax safety boundaries.
- Memory given to CodeBuddy must be filtered by Context Manager.

