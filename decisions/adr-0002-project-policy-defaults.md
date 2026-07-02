# ADR 0002: Project Policy Defaults

## Status

Accepted.

## Decision

CodeBuddy is forbidden by default from modifying these domains:

```text
auth
security
db
infra
deployment
ci/cd
```

New dependencies require approval by default.

## Rationale

The early Hermes system should optimize for stability first. CodeBuddy is useful
as a low-cost executor, but protected domains have high blast radius and are
harder to review mechanically.

Dependencies add security, maintenance, runtime, and lockfile risk. Approval
keeps this cost visible.

## Consequences

- Protected domain work requires Codex planning.
- CodeBuddy may only work in protected domains through explicitly scoped,
  approved subtasks.
- Dependency requests must include a structured approval packet.

