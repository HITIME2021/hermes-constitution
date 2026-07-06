# ADR 0007: Codex Review Modes

## Status

Accepted.

## Date

2026-07-06.

## Context

The `codex-plugin-cc` project shows a useful separation between asking Codex
for review and asking another surface to perform implementation work. Hermes
should absorb the review pattern without adopting Claude Code as a required
provider or changing the v0.2 provider boundary.

Hermes already treats Codex as the senior planner and reviewer. The missing
policy detail was when review should be a normal quality pass and when it
should be an adversarial, skeptical pass.

## Decision

Hermes defines two Codex review modes:

```text
normal
adversarial
```

`normal` review checks correctness, scope, tests, maintainability, and ordinary
quality concerns.

`adversarial` review is a stronger review profile. It actively searches for
hidden regressions, missing edge cases, unsafe scope expansion, approval
bypass, security risk, dependency risk, and provider-boundary violations.

Adversarial review is required by default for medium and higher risk tasks,
architecture or constitution changes, provider routing changes, dependency
manifest or lockfile changes, auth/security/db/infra/deployment-sensitive
changes, and repeated failure or replanning cases.

Codex review remains read-only. Review findings may request a bounded revision
or a new ExecutionRequest, but the reviewer must not silently become the
executor.

When CodeBuddy is unlikely to repair a finding correctly, or has already failed
to repair it, adversarial review may recommend `codex_scoped_repair`. This is
an execution escalation, not a review action. It requires a new
ExecutionRequest with explicit `allowed_scope`, risk evaluation, verification,
and Review Gate coverage.

## Consequences

Positive:

- Hermes gains a stronger review profile for tasks where ordinary review is
  too weak.
- The provider boundary stays clear: CodeBuddy executes, Codex reviews.
- Medium and higher risk tasks have a default skeptical review path.
- Review failures map cleanly to existing Human Intervention Policy.
- Complex findings can escalate to Codex scoped repair without weakening the
  separation between review authority and execution authority.

Limits:

- Hermes does not adopt Claude Code as a v0.2 provider.
- This ADR does not require a new plugin system.
- Codex scoped repair should remain exceptional; CodeBuddy stays the default
  low-cost scoped executor.
- Tooling may later add first-class `review_mode` fields to schemas, but the
  policy is effective immediately through Review Gate.
