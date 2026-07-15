# Constitution Release

This document tracks the human-facing Hermes constitution release line.

It is separate from:

- JSON/YAML schema versions such as `0.1.0`
- target project versions
- provider CLI versions
- git commit based `constitution_version` values in snapshots

## Current Release

<!-- snapshot:block id="constitution-release" section="Constitution Release" priority="10" -->
Current Hermes constitution release: `v0.3`.

`v0.3` means the constitution includes the validated v0.2 local orchestration
baseline and adds tool governance, artifact intake, planning-source control,
token/quality telemetry, simple shell direct-mode safety, and a stricter
Hermes self-edit boundary.

Validated v0.2 control loops:

- source-driven constitution snapshots with block extraction and index output
- Codex planning plus CodeBuddy scoped execution plus Codex review
- dependency policy based on target project dependency profiles
- low-risk production-code bugfix orchestration
- Dashboard / Kanban / Gateway lifecycle visibility for Hermes-managed tasks
- provider orchestration observability policy

Added v0.3 capabilities:

- Tools Layer classification: Frontend Tools produce artifacts; Backend Tools
  produce effects; Hermes governs both
- `planning_source_of_record`: `codex_native` or
  `frontend_artifact_assisted`
- Artifact Intake Gate before frontend artifacts influence live execution
- Token telemetry paired with quality telemetry
- Simple Shell Direct Mode snapshot coverage and hard-reject safety posture
- Hermes self-edit disabled by default; implementation work routes to
  CodeBuddy scoped execution, verification, and Codex review unless an explicit
  task-specific emergency override is given

The git commit remains the exact `constitution_version` for a loaded snapshot.
The release label is a human-facing maturity marker.
<!-- /snapshot:block -->

## Release Meaning

`v0.1` was the architecture and policy baseline.

`v0.2` is the validated local orchestration baseline for this operator setup:

```text
WSL = production execution plane
Windows = human control and constitution maintenance plane
Codex = planning and review
CodeBuddy = scoped execution
Dashboard/Kanban = observability for Hermes-managed runs
```

`v0.3` is the tool governance and artifact-intake baseline:

```text
Frontend Tools = artifact evidence, not authority
Backend Tools = authority-bearing effects with scope control
Codex native planning = default planning mode
Frontend artifact assistance = optional for complex or ambiguous work
Hermes ExecutionRequest = live execution authority
Hermes self-edit = disabled by default
```

Future release bumps should be recorded by ADR and should explain what
capability was validated, not merely that documents changed.
