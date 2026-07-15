# ADR 0009: Constitution v0.3 Release

## Status

Accepted.

## Date

2026-07-15.

## Context

Hermes constitution `v0.2` validated the local provider orchestration baseline:
source-driven snapshots, Codex planning and review, CodeBuddy scoped execution,
dependency gates, Kanban observability, and low-risk production-code smoke
tests.

Since then, the constitution added a new policy layer for tool classification
and planning artifact governance. Hermes also validated Simple Shell Direct
Mode as a policy and local implementation concern, and tightened the local
Hermes agent self-edit boundary after repeated attempts showed that self-edit
could bypass the intended provider orchestration route.

The human-facing release label must reflect this new capability set instead of
remaining at `v0.2`.

## Decision

The human-facing Hermes constitution release is bumped to `v0.3`.

`v0.3` includes the `v0.2` orchestration baseline and adds:

- Tools Layer classification:
  - Frontend Tools produce artifacts.
  - Frontend Tool output is input evidence, not authority.
  - Backend Tools produce effects.
  - Backend Tool execution is authority-bearing effect and requires scope
    control.
- Planning Modes:
  - `codex_native`
  - `frontend_artifact_assisted`
  - one `planning_source_of_record` per task
- Artifact Intake Gate for frontend artifacts before execution influence
- Token Telemetry Policy paired with quality telemetry
- Simple Shell Direct Mode snapshot coverage and hard-reject safety posture
- Hermes self-edit disabled by default, with implementation routed to
  CodeBuddy scoped execution and Codex review unless an explicit emergency
  override is given

This does not automatically change:

- schema versions such as `0.1.0`
- historical ADR titles
- historical v0.1 architecture document names
- provider CLI versions
- target project versions
- git commit based snapshot `constitution_version`

## Consequences

Positive:

- The release label now reflects the v0.3 tool governance and artifact intake
  model.
- Frontend artifact tools can be integrated without gaining execution
  authority.
- CodeBuddy remains the default scoped implementation provider.
- Hermes self-edit now has a stricter default-disabled boundary.
- Token telemetry can support optimization without replacing quality review.

Tradeoffs:

- More policy gates exist before complex tasks enter live execution.
- Artifact-assisted planning requires additional intake and review evidence.
- Local Hermes agent customizations require explicit integration approval and
  should not be confused with upstream Hermes contributions.

## Related Documents

- `docs/constitution-release.md`
- `docs/tools-layer.md`
- `docs/planning-modes.md`
- `docs/artifact-intake-gate.md`
- `docs/token-telemetry-policy.md`
- `docs/simple-shell-direct-mode.md`
- `docs/provider-adapter-design.md`
- `decisions/adr-0008-tools-layer-and-planning-source-of-record.md`
