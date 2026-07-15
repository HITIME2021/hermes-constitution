# Changelog

This changelog is for operator-facing release notes. It summarizes what changed
in the Hermes constitution release line. Detailed rationale remains in
`decisions/`; runtime policy source remains in `docs/`.

## v0.3.2 - 2026-07-15

Trusted gateway entry and self-improvement governance patch for the v0.3 line.

Added:

- Gateway Entry Guard.
- Self-Improvement Governance.
- ADR 0011 for gateway entry and self-improvement governance.

Changed:

- Human-facing constitution release label changed from `v0.3.1` to `v0.3.2`.
- Gateway / DM / mobile / webhook entrypoints are untrusted until startup
  verification succeeds.
- Non-TUI entrypoints must verify `current.md` and `current.index.json` before
  tool execution.
- Self-improvement writes are explicitly treated as authority-bearing effects.
- Gateway startup and DM cold start may generate self-improvement candidates
  but must not apply patches.

Validated:

- Gateway dry-run audit found no automatic constitution snapshot load path in
  gateway entrypoint.
- Background self-improvement review can patch skills after nudge intervals;
  this is now governed as candidate-first behavior for untrusted entrypoints.

## v0.3.1 - 2026-07-15

Workspace layout and tool-validation patch for the v0.3 line.

Added:

- Workspace Layout Policy:
  - `~/projects/production`
  - `~/projects/labs`
  - `~/projects/worktrees`
  - `~/projects/archive`
- ADR 0010 for the v0.3.1 workspace layout decision.
- Validation records:
  - Run-SPEC-001: spec-kit Tools Adapter smoke test.
  - Run-SPEC-002: spec-kit Artifact Intake Gate smoke test.

Changed:

- Human-facing constitution release label changed from `v0.3` to `v0.3.1`.
- Tools Adapter now distinguishes `project-read-only` from `environment no-effect`.
- `uvx`-based tool invocation must record possible network/cache/runtime effects.
- Tool evidence intended for audit or long-term operator review should be UTF-8.

Validated:

- `spec-kit/specify` can be invoked through the generic Tools Adapter.
- `specify check` output is environment evidence only and does not change Provider Policy.
- `specify init` is a backend effect and requires bounded scope plus operator approval.
- spec-kit artifacts can enter Artifact Intake Gate as input evidence.
- Hidden mutating shell steps in artifacts can be detected and marked.

## v0.3 - 2026-07-15

Tool governance and artifact-intake baseline.

Added:

- Tools Layer.
- Tools Adapter.
- Planning Modes and `planning_source_of_record`.
- Artifact Intake Gate.
- Token Telemetry Policy paired with quality telemetry.
- Prompt Distillation policy.
- Session Startup Policy.
- Simple Shell Direct Mode snapshot coverage.
- Provider self-edit cost boundary.
- Constitution v0.3 release ADR.
- Hermes v0.3 flow diagram.

Changed:

- Hermes self-edit is disabled by default.
- Implementation work routes to CodeBuddy scoped execution by default.
- Codex remains the default planning, review, and adversarial review provider.
- Frontend Tool output is input evidence, not authority.
- Backend Tool execution is authority-bearing effect and requires scope control.

Validated:

- Provider orchestration dry-run/live-run patterns.
- Codex planning -> CodeBuddy execution -> pytest -> Codex review loop.
- Kanban/Dashboard run observability.
- Memory writeback approval boundary.
- Tool classification and Artifact Intake behavior.

## v0.2

Validated local orchestration baseline.

Added:

- Source-driven constitution snapshot generation.
- Kanban/Dashboard observability policy.
- Human intervention stop conditions.
- Provider orchestration evidence requirements.
- Dependency profile based stop condition handling.

Validated:

- Low-risk test append.
- README/docs correction.
- Small CLI behavior fix.
- Report display bugfix.
- Review and repeated-finding stop behavior.

## v0.1

Architecture and policy baseline.

Added:

- Hermes role and provider model.
- Codex as planning/review brain.
- CodeBuddy as scoped execution worker.
- Project Policy defaults.
- Workflow state machine.
- Capability resolver.
- Context manager.
- Review gate.
- Memory center.
