# Constitution Release

This document tracks the human-facing Hermes constitution release line.

It is separate from:

- JSON/YAML schema versions such as `0.1.0`
- target project versions
- provider CLI versions
- git commit based `constitution_version` values in snapshots

## Current Release

<!-- snapshot:block id="constitution-release" section="Constitution Release" priority="10" -->
Current Hermes constitution release: `v0.4.0`.

`v0.4.0` means the constitution includes the validated v0.2 local orchestration
baseline, the v0.3 tool governance and artifact-intake line, the v0.3.1
workspace layout patch, and the v0.3.2 trusted-entry/self-improvement
governance patch. It further adds local background model governance: Ollama may
be used as a bounded text-processing layer, but never as planning, review,
execution, approval, memory, or constitution authority.

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
- Workspace Layout Policy separates `~/projects/production`, `~/projects/labs`,
  `~/projects/worktrees`, and `~/projects/archive`
- Gateway Entry Guard requires non-TUI entrypoints to verify `current.md` and
  `current.index.json` before tool execution
- Self-Improvement Governance allows automatic candidate generation but treats
  patch application as an authority-bearing effect
- Background Local Model Adapter allows local Ollama text preprocessing with
  `output_authority: none`, a deterministic context budget gate, no tool
  calling, proxy bypass for local traffic, and Hermes Primary validation
- Ollama dispatch integration supports explicit marker routing and a
  deterministic auto classifier behind environment gates; it remains disabled
  by default

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

`v0.3.1` is the workspace layout patch for the v0.3 line:

```text
production repos -> ~/projects/production
tool validation and smoke projects -> ~/projects/labs
provider worktrees -> ~/projects/worktrees
retained old experiments -> ~/projects/archive
```

`v0.3.2` is the trusted-entry and self-improvement governance patch:

```text
gateway / DM / mobile entrypoints -> untrusted until startup verification
current.md + current.index.json -> required before tool execution
self-improvement analysis -> allowed
self-improvement patch application -> trusted channel + scope + approval
```

`v0.4.0` is the local background model governance and Ollama dispatch
validation line:

```text
Ollama/local model -> bounded text preprocessing only
output authority -> none
context budget -> checked before local model dispatch
explicit marker dispatch -> validated
deterministic auto classifier -> validated behind double env gate
planning/review/execution/approval/memory/scope -> bypass Ollama
```

Future release bumps should be recorded by ADR and should explain what
capability was validated, not merely that documents changed.
