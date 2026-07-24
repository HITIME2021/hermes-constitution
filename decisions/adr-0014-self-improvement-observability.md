# ADR 0014: Self-Improvement Observability

Status: Proposed

Date: 2026-07-24

## Context

Hermes self-improvement is a core learning mechanism. It lets Hermes improve
skills, prompts, profiles, paths, and workflow guidance after repeated operator
corrections or observed failures.

A self-improvement audit reported:

- 284 applied patches across 16 skills.
- 0 operator-visible patch records.
- 3 high-risk skills accounting for 187 patches, or about 66 percent of the
  observed patch volume.
- 6 evidence gaps. The most severe confirmed gaps were no git-backed version
  control for the patched skill state and no durable diff audit trail.

The issue is not that Hermes learns. The issue is that useful self-improvement
patches can become silent authority if they are not attributable, diffable,
classified, visible, and reversible.

## Decision

Keep self-improvement enabled as a Hermes learning mechanism, but govern every
applied patch with explicit observability requirements.

Every applied self-improvement patch should produce a patch record containing:

- source candidate or trigger
- target skill or file
- before and after identity
- visible diff path
- risk class
- operator visibility state
- approval requirement
- rollback hint
- verification evidence

Risk classes are:

- `low`: wording, typo, stale path, documentation, or non-behavioral skill
  clarification.
- `medium`: workflow ordering, default behavior, tool classification, report
  format, or policy wording that can change routing decisions.
- `high`: approval gates, scope boundaries, provider routing, memory,
  credentials, auth, shell/tool execution, runtime code, dependency manifests,
  or any change that can grant execution authority.

High-risk self-improvement requires explicit operator approval before
application. If an existing runtime applies a high-risk patch before approval,
the patch must be quarantined for review before future runs treat it as trusted
behavior.

Long self-improvement reports should be written to local evidence files. Chat
responses should return the final status, critical findings, decisions needed,
telemetry or token summary when available, and local inspection commands rather
than dumping the entire report by default.

## Consequences

- Hermes keeps its self-improvement capability.
- Self-improvement becomes auditable and reversible.
- Gateway, DM, background review, and Kanban paths can surface candidates and
  reports without silently granting execution authority.
- Operators can inspect local reports with `cat`, `less`, or `rg` when the full
  report is too long for chat.

## Rejected Alternatives

- Disable self-improvement entirely.
  - Rejected because self-improvement is one of Hermes's defining mechanisms.
- Trust all self-improvement patches if tests pass.
  - Rejected because tests do not prove authority, scope, rollback, or
    provenance.
- Require manual editing for every low-risk wording patch.
  - Rejected because that would remove useful learning for low-risk
    corrections. Low-risk automation is acceptable when it is visible,
    diffable, and reversible.
