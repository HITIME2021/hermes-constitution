# ADR 0013: Constitution v0.4.1 Snapshot Layering

Status: Proposed

Date: 2026-07-17

## Context

Hermes v0.4.0 added local background model governance, gateway entry guards,
tools/artifact governance, and more validation records. The generated
constitution snapshot is becoming longer. Keeping one full `current.md` as the
only runtime artifact increases context cost and makes gateway/mobile entry
startup heavier.

Earlier policy correctly stated that snapshot length is a soft optimization and
that required policy must not be deleted to hit a line target. The next step is
therefore not to compress policy away, but to split generated snapshot output
into loadable layers.

## Decision

Hermes should adopt a layered snapshot model for the v0.4.1 line:

```text
boot.md       -> startup trust, version, index, load order, fallback
core.md       -> always-needed runtime policy
packs/*.md    -> domain policy loaded on demand
current.md    -> full compatibility snapshot
```

`current.md` remains the compatibility snapshot and must continue to be
generated from the same source block set. Existing entrypoints that only know
`current.md` remain valid.

Layer-aware entrypoints should load:

1. `boot.md` and `current.index.json`
2. `core.md`
3. deterministic domain packs required by the task, command, entrypoint, or
   adapter

Pack selection is policy-bearing and must be deterministic. A model may suggest
pack choices as draft text, but Hermes must compute the final pack set.

## Required Domain Packs

Initial packs:

- `commands.md`
- `gateway.md`
- `local-models.md`
- `memory.md`
- `project-policy.md`
- `provider-adapters.md`
- `review-and-execution.md`
- `tools-and-artifacts.md`
- `workspace.md`

The pack names are stable generated artifact names, not source-document names.

## Compatibility

`current.index.json` remains the single generated index file. It should add
optional `snapshot_layout` metadata while preserving existing fields:

- `constitution_version`
- `generated_at`
- `source_repo`
- `snapshot_path`
- `blocks`
- `changes`
- `diagnostics`

Older loaders may ignore `snapshot_layout` and continue loading `current.md`.

## Safety

Layered loading must fail closed:

- missing or invalid `boot.md` -> `UNTRUSTED_CONTEXT_STOP`
- missing or invalid `current.index.json` -> `UNTRUSTED_CONTEXT_STOP`
- missing relevant pack -> valid `current.md` fallback or
  `UNTRUSTED_CONTEXT_STOP`
- version mismatch across boot/core/pack/current/index ->
  `UNTRUSTED_CONTEXT_STOP`

Layering must not allow partial policy execution when the missing layer is
relevant to the current task.

## Consequences

Positive:

- lower routine context size
- better gateway/DM cold-start behavior
- explicit policy domains
- easier future pack-level validation and cache invalidation

Costs:

- snapshot generator complexity increases
- index schema needs optional layer metadata
- gateway/CLI loaders need a staged implementation
- tests must cover fallback and fail-closed behavior

## Implementation Order

1. Document the layering policy and index metadata.
2. Extend `/reload-constitution` to emit `boot.md`, `core.md`, and
   `packs/*.md` while preserving `current.md`.
3. Extend `/load-constitution-snapshot` to support `current_compat`,
   `boot_core`, and `boot_core_domain` modes.
4. Add gateway/DM startup verification for `boot.md` plus index.
5. Add deterministic pack selection and tests.
6. After validation, mark v0.4.1 as the human-facing release patch.

## Non-Goals

- Removing `current.md`.
- Treating generated files as source of truth.
- Letting models choose policy packs as authority.
- Compressing required policy out of the snapshot.
