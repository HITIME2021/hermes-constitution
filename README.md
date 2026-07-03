# Hermes Constitution

Chinese overview: [README.zh-CN.md](README.zh-CN.md)

This repository contains the working constitution for Hermes v0.1.

Hermes is designed as a multi-agent automation platform. Its core job is not to
write every line of code itself, but to orchestrate roles, skills, context,
execution providers, review gates, and long-term memory.

## Current v0.1 Position

- Hermes is the orchestration and learning layer.
- Codex / GPT-5.5 is the senior brain: architecture, planning, algorithm design,
  risk evaluation, review, replanning, and memory synthesis.
- CodeBuddy is the low-cost execution worker: scoped implementation, repetitive
  code edits, small bug fixes, and test writing.
- Claude Code is excluded from v0.1.
- Default operating mode is `stable`.
- `quality` mode is preserved as a stronger review and planning profile.

## Main Flow

```text
User Intent
  -> Intake / Clarification
  -> Workflow Engine
  -> Task Planner
  -> Risk Evaluator
  -> Agency Registry
  -> Capability Resolver
  -> Provider Routing
  -> Execution
  -> Review Gate
  -> Testing / Verification
  -> Finalize
  -> Memory Writeback
```

The central routing idea is:

```text
task -> role -> skill -> provider -> review -> memory
```

## Directory Map

```text
docs/
  Architecture and module design documents.

schemas/
  Draft YAML schemas for core Hermes objects.

decisions/
  Architecture decision records.

diagrams/
  Mermaid diagrams.
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
- Normal sessions should load `~/hermes-snapshots/current.md` as a pinned
  constitution snapshot instead of rereading the full constitution repository.
- User slash commands should be implemented through Command Handlers with
  declared `effects`, `no_effects`, and a Command Policy Gate.
- Codex and CodeBuddy should be integrated through provider adapters over
  CLI/API for automation.
- Project Policy has priority over AgentProfile, Skill, and Provider preference.
- CodeBuddy is forbidden by default from modifying `auth`, `security`, `db`, and
  `infra` domains.
- New dependencies require approval by default.
- Medium and higher risk tasks require Codex planning or Codex review.

## Resume On Another Machine

On the Windows 11 + WSL machine that runs the Hermes agent, copy or pull this
repository and point Hermes at these documents as its v0.1 constitution.

Recommended future flow:

```bash
git clone <repo-url>
cd "Hermes Constitution"
```

Then use the files in `docs/`, `schemas/`, and `decisions/` as the design source
for implementation.
