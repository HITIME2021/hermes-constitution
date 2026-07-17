# Snapshot Layering Policy

Snapshot layering is the planned v0.4.1 answer to constitution snapshot growth.
It reduces routine prompt size without weakening startup trust, policy
attribution, or backwards compatibility.

The full `hermes-constitution` repository remains the source of truth. Layered
snapshots are generated cache artifacts under `~/hermes-snapshots/`.

## Layered Artifacts

Generated artifacts:

```text
~/hermes-snapshots/
  boot.md
  core.md
  current.md
  current.index.json
  packs/
    commands.md
    gateway.md
    local-models.md
    memory.md
    project-policy.md
    provider-adapters.md
    review-and-execution.md
    tools-and-artifacts.md
    workspace.md
  archive/
    constitution-<commit>-<date>.md
```

`current.md` remains the full compatibility snapshot. It must continue to be
generated until all Hermes entrypoints can load layered snapshots natively.

`boot.md` is the minimal trust and routing layer. It must be small, stable, and
safe to load at every entrypoint before provider calls or tool execution.

`core.md` is the always-loaded runtime policy layer. It contains the policies
that are needed for almost every task.

`packs/*.md` are domain packs. They are loaded only when the current task,
command, entrypoint, or adapter needs that policy domain.

## Boot Layer

<!-- snapshot:block id="snapshot-layering-boot" section="Constitution Snapshot" priority="32" -->
Layered snapshot startup must begin with `boot.md` plus `current.index.json`.

`boot.md` is not sufficient execution authority. It exists to verify trust,
identify the loaded constitution, select required policy layers, and define the
fallback path.

Required boot fields:

```yaml
constitution_version: <git commit>
release: <human-facing release>
source_repo: ~/projects/production/hermes-constitution
index_path: ~/hermes-snapshots/current.index.json
compat_snapshot_path: ~/hermes-snapshots/current.md
core_snapshot_path: ~/hermes-snapshots/core.md
pack_dir: ~/hermes-snapshots/packs
index_hash: sha256:<hash>
load_order:
  - boot
  - core
  - selected_domain_packs
fallback:
  compatible_current_md: true
```

If `boot.md` or `current.index.json` is missing, unreadable, malformed, or
hash-inconsistent, Hermes must stop with `UNTRUSTED_CONTEXT_STOP` before any
provider call, tool call, shell command, memory writeback, self-improvement
action, or task execution.
<!-- /snapshot:block -->

## Core Layer

<!-- snapshot:block id="snapshot-layering-core" section="Constitution Snapshot" priority="33" -->
`core.md` must be loaded for normal agent work after `boot.md` succeeds.

Core policy should include:

- constitution version attribution
- session startup and reload conditions
- gateway entry guard and stale-session invalidation
- workspace layout
- command handler effect model
- project policy precedence
- provider adapter boundary and no-default-self-edit rule
- planning, execution, review, and stop-condition gates
- memory writeback approval boundary
- token and quality telemetry requirements
- simple shell direct-mode safety summary
- local background model authority boundary

`core.md` should not attempt to include every specialized rule. Domain-specific
details belong in packs and must be loaded when the task touches that domain.
<!-- /snapshot:block -->

## Domain Packs

<!-- snapshot:block id="snapshot-layering-domain-packs" section="Constitution Snapshot" priority="34" -->
Domain packs are generated policy slices loaded on demand.

Initial pack map:

| Pack | Load When |
|------|-----------|
| `commands.md` | slash command handling, reload/load snapshot, command effects |
| `gateway.md` | gateway, DM, mobile, webhook, pairing, entry trust |
| `local-models.md` | Ollama/local background model routing or context budget |
| `memory.md` | MemoryCandidate, writeback, memory synthesis |
| `project-policy.md` | dependency, auth, security, db, infra, target repo scope |
| `provider-adapters.md` | Codex, CodeBuddy, Hermes Primary, provider confirmation |
| `review-and-execution.md` | dry-run, ExecutionRequest, stop conditions, Codex review |
| `tools-and-artifacts.md` | Tools Layer, Tools Adapter, Artifact Intake Gate |
| `workspace.md` | production/labs/worktrees/archive layout |

Pack selection must be deterministic and auditable. A model may suggest a pack
set as draft text, but Hermes must compute the final pack set from command
type, entrypoint, task classification, declared effects, and explicit operator
request.

When a selected pack is missing, unreadable, hash-invalid, or version-mismatched,
Hermes must either fall back to a valid `current.md` compatibility snapshot or
stop with `UNTRUSTED_CONTEXT_STOP`. It must not proceed using partial policy
when the missing pack is relevant to the task.
<!-- /snapshot:block -->

## Compatibility Snapshot

<!-- snapshot:block id="snapshot-layering-current-compat" section="Constitution Snapshot" priority="35" -->
`current.md` remains the compatibility snapshot.

Compatibility rules:

- existing `/load-constitution-snapshot ~/hermes-snapshots/current.md` behavior
  remains valid
- gateway entry guards may continue to verify `current.md` and
  `current.index.json`
- `current.md` must be generated from the same block set as the layered files
- `current.md` must not become an independent policy source or hand-maintained
  template
- layered loaders should prefer `boot.md` + `core.md` + selected packs when
  supported, and fall back to `current.md` only as a compatibility or recovery
  path

The release label and `constitution_version` must match across `boot.md`,
`core.md`, selected packs, `current.md`, and `current.index.json`.
<!-- /snapshot:block -->

## Index Metadata

`current.index.json` should remain backwards compatible while adding optional
layer metadata.

Recommended additions:

```json
{
  "snapshot_layout": {
    "mode": "layered_with_current_compat",
    "boot_path": "~/hermes-snapshots/boot.md",
    "core_path": "~/hermes-snapshots/core.md",
    "compat_snapshot_path": "~/hermes-snapshots/current.md",
    "pack_dir": "~/hermes-snapshots/packs",
    "packs": [
      {
        "id": "local-models",
        "path": "~/hermes-snapshots/packs/local-models.md",
        "sections": ["Tools Layer", "Provider Adapter"],
        "block_ids": ["background-local-model-adapter"],
        "hash": "sha256:..."
      }
    ]
  }
}
```

Older loaders may ignore `snapshot_layout` and continue using `current.md`.

## Load Modes

```text
boot_only
  -> startup verification only; not enough for task execution

boot_core
  -> normal lightweight session baseline

boot_core_domain
  -> core plus selected packs for the current task

current_compat
  -> full current.md fallback for old loaders or recovery
```

Task reports should state the effective load mode when policy matters.

## Generator Rules

`/reload-constitution` should eventually:

- scan source `snapshot:block` declarations once
- render `boot.md`
- render `core.md`
- render domain packs
- render full `current.md` from the same block set
- write one `current.index.json` containing block hashes and layer metadata
- archive the compatibility snapshot and optionally archive layer artifacts
- report added, changed, removed, duplicate, malformed, omitted, and
  pack-assignment changes

The generator must not use previous generated files as policy authority.

## Non-Goals

Snapshot layering does not:

- change the constitution source of truth
- remove `current.md`
- allow model-selected policy packs to become authority
- permit partial policy execution when a relevant pack is missing
- replace Codex review, CodeBuddy scoped execution, operator approval, or
  Gateway Entry Guard
