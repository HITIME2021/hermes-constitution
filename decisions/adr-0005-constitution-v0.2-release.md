# ADR 0005: Constitution v0.2 Release

## Status

Accepted.

## Date

2026-07-06.

## Context

The Hermes constitution started as a v0.1 architecture and policy baseline.
Since then, the local operator setup has validated multiple practical control
loops:

- source-driven constitution snapshots
- Codex planning and review
- CodeBuddy scoped execution
- dependency profile based policy checks
- low-risk production-code bugfix orchestration
- Dashboard, Kanban, and Gateway task lifecycle visibility
- provider orchestration observability policy

Keeping the human-facing release label at v0.1 no longer reflects the current
state of the constitution.

## Decision

The human-facing Hermes constitution release is bumped to `v0.2`.

This does not automatically change:

- schema versions such as `0.1.0`
- historical ADR titles
- historical v0.1 architecture document names
- provider CLI versions
- target project versions
- git commit based snapshot `constitution_version`

## Consequences

Positive:

- The release label now reflects the validated local orchestration baseline.
- Future release bumps have a precedent: they should follow capability
  validation, not document churn.

Tradeoffs:

- Some historical documents still mention v0.1 because they describe the
  original baseline.
- Schema versions remain independent and should be bumped only when their
  machine contract changes.
