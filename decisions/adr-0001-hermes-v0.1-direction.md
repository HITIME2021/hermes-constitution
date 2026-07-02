# ADR 0001: Hermes v0.1 Direction

## Status

Accepted.

## Context

Hermes is intended to become a multi-agent automation platform that can schedule
work across multiple execution providers while learning from experience.

The initial design prioritizes a practical two-provider model:

- Codex / GPT-5.5 as the senior reasoning brain.
- CodeBuddy as the low-cost scoped executor.

## Decision

Hermes v0.1 will use `stable` mode by default.

The main execution strategy is:

```text
Codex plans and reviews.
CodeBuddy executes scoped implementation work.
Hermes controls workflow, policy, context, memory, and review gates.
```

Claude Code is excluded from v0.1 to keep the initial architecture simple.

## Consequences

Positive:

- cheaper execution for scoped work
- stronger control over risk
- cleaner implementation boundary
- easier debugging of orchestration behavior

Tradeoffs:

- fewer provider options in v0.1
- more responsibility on Codex as planner and reviewer
- CodeBuddy requires clear task packets and strict policy boundaries

