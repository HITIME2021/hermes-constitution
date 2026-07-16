# Changelog

This changelog is for operator-facing release notes. It summarizes what changed
in the Hermes constitution release line. Detailed rationale remains in
`decisions/`; runtime policy source remains in `docs/`.

## v0.4.0 - 2026-07-16

Local background model governance and Ollama dispatch validation.

Added:

- ADR 0012 for local background model governance.
- Hermes v0.4 governed orchestration flow diagram:
  `diagrams/hermes-v0.4-flow.mmd`.
- Local Hermes runtime patch group reference:
  `docs/local-hermes-runtime-patches.md`.
- Ollama process-boundary incident record:
  `docs/validation/ollama-process-boundary-incident.md`.

Changed:

- Human-facing constitution release label changed from `v0.3.2` to `v0.4.0`.
- Ollama/local models are governed as background text-processing helpers only.
- Ollama output remains `output_authority: none` and must not become planning,
  review, execution, approval, memory, or constitution authority.
- Automatic Ollama routing remains disabled by default and requires both
  `HERMES_OLLAMA_BACKGROUND_TEXT=1` and
  `HERMES_OLLAMA_BACKGROUND_TEXT_AUTO=1`.
- Automatic routing is limited to high-confidence text preprocessing:
  `evidence_summary`, `translation`, `compression`, and
  `text_normalization`.

Validated:

- OLLAMA-003: adapter process-boundary enforcement, 56 tests, Codex pass.
- OLLAMA-004A: adapter live cost smoke, 5/5 cases pass, estimated 4,700
  DeepSeek tokens saved in the smoke batch.
- OLLAMA-004B: explicit-purpose dispatch hook, 76 tests, Codex pass.
- OLLAMA-004C: deterministic auto classifier, 49 tests, Codex pass.
- Total OLLAMA line evidence: 143 tests reported, six Codex review rounds,
  risk after review LOW.

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
- Added draft Background Local Model Adapter policy for Ollama-style local text
  processing, including a context budget gate for `qwen2.5:14b`.
- Added draft Hermes Primary Adapter Boundary policy after a context-budget
  helper process incident showed that low-risk tasks can still bypass dry-run
  and approval gates if Hermes Primary self-edits.

Validated:

- Gateway dry-run audit found no automatic constitution snapshot load path in
  gateway entrypoint.
- Background self-improvement review can patch skills after nudge intervals;
  this is now governed as candidate-first behavior for untrusted entrypoints.
- Weixin DM smoke testing found that persisted DM session history can override
  current startup verification in model behavior; `/new` session reset restored
  the current `constitution_version` and this is recorded in
  `docs/validation/gateway-dm-session-history-smoke-test.md`.
- Context budget helper review found safe code but a medium-severity process
  violation where the expected dry-run and approval gate was bypassed; recorded
  in `docs/validation/context-budget-helper-process-incident.md`.
- Ollama background text adapter validation passed across text-only use,
  deterministic context gating, proxy bypass, and authority-boundary review;
  recorded in `docs/validation/ollama-background-text-adapter-smoke-test.md`.
- The Ollama validation line now includes OLLAMA-003 process-boundary
  enforcement, OLLAMA-004A adapter live cost smoke, OLLAMA-004B explicit-purpose
  dispatch integration, and OLLAMA-004C deterministic auto-classification. The
  local model remains disabled by default; automatic routing requires a double
  env gate and is limited to high-confidence text preprocessing.
- Local Hermes runtime `.py` changes are now documented as reference patch
  groups in `docs/local-hermes-runtime-patches.md`, so they can be audited and
  selectively reused without treating them as upstream Hermes Agent source.
- Ollama/background model process-boundary violations are now recorded
  separately from code safety in
  `docs/validation/ollama-process-boundary-incident.md`; this prevents the
  v0.4.0 line from treating passing tests as sufficient when dry-run,
  approval, or review gates were bypassed.

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
