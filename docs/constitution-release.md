# Constitution Release

This document tracks the human-facing Hermes constitution release line.

It is separate from:

- JSON/YAML schema versions such as `0.1.0`
- target project versions
- provider CLI versions
- git commit based `constitution_version` values in snapshots

## Current Release

<!-- snapshot:block id="constitution-release" section="Constitution Release" priority="10" -->
Current Hermes constitution release: `v0.2`.

`v0.2` means the constitution has moved beyond the initial v0.1 design baseline
and has validated the first practical control loops:

- source-driven constitution snapshots with block extraction and index output
- Codex planning plus CodeBuddy scoped execution plus Codex review
- dependency policy based on target project dependency profiles
- low-risk production-code bugfix orchestration
- Dashboard / Kanban / Gateway lifecycle visibility for Hermes-managed tasks
- provider orchestration observability policy

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

Future release bumps should be recorded by ADR and should explain what
capability was validated, not merely that documents changed.
