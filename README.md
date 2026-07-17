# Hermes Constitution

Chinese overview: [README.zh-CN.md](README.zh-CN.md)

This repository contains the working constitution for Hermes v0.4.0.

Hermes is designed as a multi-agent automation platform. Its core job is not to
write every line of code itself, but to orchestrate roles, skills, context,
execution providers, review gates, and long-term memory.

## Current v0.4.0 Position

- Hermes is the orchestration and learning layer.
- Codex / GPT-5.5 is the senior brain: architecture, planning, algorithm design,
  risk evaluation, review, replanning, and memory synthesis.
- CodeBuddy is the low-cost execution worker: scoped implementation, repetitive
  code edits, small bug fixes, and test writing.
- Ollama/local background models are bounded text-processing helpers for
  summaries, translations, compression, evidence cleanup, and draft text. They
  are not authority surfaces.
- Claude Code is excluded from v0.1.
- Default operating mode is `stable`.
- `quality` mode is preserved as a stronger review and planning profile.

## Main Flow

```text
User Intent
  -> Session Snapshot Load
  -> Intake / Clarification
  -> Simple Shell Direct Mode? / Command Handler?
  -> Tools Layer Classification
  -> Tools Adapter Invocation Plan
  -> Artifact Intake Gate, when frontend artifacts are present
  -> Planning Source Selection
  -> Optional Local Background Text Layer
     (explicit marker or deterministic auto classifier;
      Ollama output is draft-only and original input is preserved)
  -> Task / ExecutionRequest / ReviewPlan
  -> Risk, Scope, Dependency, Auth, and Stop-Condition Gates
  -> Operator Approval, when required
  -> Provider Routing
  -> Scoped Execution
  -> Verification
  -> Codex Review Gate
  -> Run Observability / Evidence / Token and Quality Telemetry
  -> Final Report
  -> MemoryCandidate or approved Memory Writeback
```

The central routing idea is:

```text
intent
  -> command or tool classification
  -> artifact intake or native planning
  -> optional non-authoritative background text preprocessing
  -> Task / ExecutionRequest / ReviewPlan
  -> provider execution
  -> verification
  -> review
  -> evidence
  -> memory decision
```

Frontend Tool output is input evidence, not authority. Backend Tool execution
is authority-bearing effect and requires scope control. Hermes remains the
execution authority through `ExecutionRequest`, review gates, stop conditions,
and operator approval.

## Directory Map

```text
docs/
  Architecture and module design documents.
  - tools-layer.md: Frontend/Backend tool classification and intake rules.
  - tools-adapter.md: Generic invocation, scope, evidence, and artifact mapping contract for external tools.
  - background-local-model-adapter.md: Ollama/local model text-processing boundary and context budget gate.
  - snapshot-layering.md: Planned v0.4.1 boot/core/domain-pack snapshot model.
  - hermes-primary-adapter-boundary.md: Default no-self-edit boundary for Hermes Primary.
  - local-hermes-runtime-patches.md: Reference map for local Hermes Agent runtime patch groups.
  - workspace-layout-policy.md: WSL workspace separation for production, labs, worktrees, and archives.
  - gateway-entry-guard.md: Trust checks for gateway, DM, mobile, and webhook entrypoints.
  - self-improvement-governance.md: Candidate-first governance for Hermes self-improvement.
  - planning-modes.md: Planning source of record and Codex role by mode.
  - artifact-intake-gate.md: Artifact validation, normalization, and mapping.
  - token-telemetry-policy.md: Token/cost telemetry paired with quality signals.
  - prompt-distillation.md: Prompt compression and provider packet policy.
  - session-startup-policy.md: Snapshot loading and reload rules.

schemas/
  Draft YAML schemas for core Hermes objects.

decisions/
  Architecture decision records.

diagrams/
  Mermaid diagrams.
  - hermes-v0.4-flow.mmd: Current v0.4 governed orchestration flow.
  - hermes-v0.3-flow.mmd: Historical v0.3 governed orchestration flow.
  - hermes-v0.1-flow.mmd: Historical v0.1 baseline flow.
```

## Important Decisions

- Roles come from `msitarzewski/agency-agents`, but Hermes uses an adapter to
  normalize them into Hermes `AgentProfile` objects.
- `garrytan/gstack` is useful as workflow and skill inspiration, especially for
  Think -> Plan -> Build -> Review -> Test -> Ship -> Reflect.
- Hermes core is headless. CLI/API is the primary automation interface. GUI is
  an optional control plane for humans.
- In the Windows 11 + WSL operator setup, WSL is the production execution plane.
  Windows is the human control and constitution maintenance plane.
- Codex CLI and CodeBuddy CLI should be installed and invoked in WSL when the
  production repository, Python/uv, Node.js, tests, and package managers live
  there.
- Human-facing output and long-term memory should default to Simplified Chinese
  for this operator environment. Protocol keys, schema fields, commands, paths,
  and code symbols remain in their original language.
- DeepSeek-V4-Pro may be configured as an LLM model backend, but it is not a
  formal Hermes provider in v0.1. It cannot replace Codex planning/review or
  CodeBuddy execution unless Project Policy explicitly promotes it.
- Codex CLI should use the operator's WSL ChatGPT/OAuth session by default.
  Hermes must not inject `OPENAI_API_KEY` or inspect credential material unless
  explicitly approved.
- Normal sessions should load `~/hermes-snapshots/current.md` as a pinned
  constitution snapshot instead of rereading the full constitution repository.
- `/reload-constitution` should generate snapshots from declared source
  `snapshot:block` markers and write `current.index.json`; it must not depend on
  a hand-maintained template as the policy source of truth.
- The planned v0.4.1 snapshot layering model keeps `current.md` as a full
  compatibility snapshot while adding generated `boot.md`, `core.md`, and
  `packs/*.md` artifacts for lower-cost runtime loading.
- User slash commands should be implemented through Command Handlers with
  declared `effects`, `no_effects`, and a Command Policy Gate.
- Dashboard and Kanban may be used as observability surfaces for
  Hermes-managed provider orchestration runs, but they are not execution
  authority and must not bypass policy or approval gates.
- Safe high-frequency read-only shell inspection commands may use Simple Shell
  Direct Mode to avoid unnecessary planning and token usage.
- Long prompts should be distilled into structured Task, ExecutionRequest,
  ReviewPlan, stop-condition, and evidence packets before provider dispatch.
- Tools must be classified by effect: Frontend Tools produce advisory
  artifacts, Backend Tools produce effects, and Hermes governs both. Ambiguous
  tools default to Backend Tool treatment until the current usage mode is
  proven artifact-only.
- Hermes must choose one planning source of record per task:
  `codex_native` by default, or `frontend_artifact_assisted` for complex or
  ambiguous work that benefits from frontend planning artifacts.
- Frontend artifacts must pass Artifact Intake Gate before they influence live
  execution: validate source and tool mode, classify artifact type, detect
  hidden effects, map to Hermes-native drafts, and pass review/approval.
- Token telemetry should be recorded per provider/tool phase when available,
  but it must be paired with quality telemetry; Hermes must not fabricate token
  counts or optimize for low token use alone.
- Ollama/local background models may transform bounded text packets, but they
  are not planning, review, execution, approval, memory, or constitution
  authority. Over-budget context must be routed back to Hermes Primary.
- The v0.4.0 local background model line has validated adapter
  process-boundary enforcement, explicit-purpose dispatch, and deterministic
  auto-classification. Ollama remains disabled by default and automatic routing
  requires both `HERMES_OLLAMA_BACKGROUND_TEXT=1` and
  `HERMES_OLLAMA_BACKGROUND_TEXT_AUTO=1`.
- Automatic Ollama routing is limited to high-confidence text preprocessing:
  `evidence_summary`, `translation`, `compression`, and
  `text_normalization`. Planning, review, approval, execution, memory,
  stop-condition, scope-expansion, dependency, auth, git, database, and infra
  work bypasses Ollama.
- Ollama/local model validation must record process violations separately from
  code safety. Passing local adapter tests does not excuse bypassed dry-run,
  approval, scoped execution, or review gates.
- Local Hermes runtime `.py` patches should be documented as reference patch
  groups, not as upstream Hermes Agent source. Other operators may adapt them
  selectively after comparing against their own Hermes version and rerunning
  review.
- Local Ollama traffic must bypass workstation HTTP proxies; `127.0.0.1`,
  `localhost`, and `::1` should be present in `NO_PROXY` / `no_proxy` before
  diagnosing Ollama `503` responses as model failures.
- Hermes Primary is the orchestrator, not the default coder. Implementation-like
  work must default to dry-run planning, operator approval, CodeBuddy scoped
  execution, verification, and Codex review unless the operator grants an
  explicit task-specific self-edit override.
- New Hermes sessions should load `~/hermes-snapshots/current.md` by default;
  full constitution reload is used only when reload conditions are met.
- Hermes must stop automatic loops and request human intervention after bounded
  retry, revision, or replanning budgets are exhausted.
- The human-facing constitution release is `v0.4.0`; git commit based
  `constitution_version` remains the exact snapshot version.
- New WSL projects should be separated by workspace class:
  `~/projects/production`, `~/projects/labs`, `~/projects/worktrees`, and
  `~/projects/archive`.
- Codex and CodeBuddy should be integrated through provider adapters over
  CLI/API for automation.
- Hermes self-edit is disabled by default. Implementation work should route to
  CodeBuddy scoped execution, followed by verification and Codex review when
  risk requires it. Emergency self-edit requires explicit task-specific
  operator override.
- Gateway / DM / mobile entrypoints are untrusted until they verify the loaded
  constitution snapshot and index.
- Self-improvement may generate candidates automatically, but applying patches
  is an authority-bearing effect that requires a trusted channel and approval.
- Project Policy has priority over AgentProfile, Skill, and Provider preference.
- CodeBuddy is forbidden by default from modifying `auth`, `security`, `db`, and
  `infra` domains.
- New dependencies require approval by default.
- Medium and higher risk tasks require Codex planning or Codex review.

## Resume On Another Machine

On the Windows 11 + WSL machine that runs the Hermes agent, copy or pull this
repository and point Hermes at these documents as its v0.4.0 constitution.

Recommended future flow:

```bash
git clone <repo-url>
cd "Hermes Constitution"
```

Then use the files in `docs/`, `schemas/`, and `decisions/` as the design source
for implementation.
