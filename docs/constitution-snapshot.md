# Constitution Snapshot Policy

Constitution snapshots are compact, human-readable summaries of the current
Hermes constitution. They reduce repeated full-document loading and lower
context cost during normal operation.

The full constitution repository remains the source of truth. A snapshot is a
cache pinned to a constitution commit.

## Storage Policy

Generated snapshots must not be stored in `hermes-constitution`.

Default local storage:

```yaml
constitution_snapshot:
  default_path: ~/hermes-snapshots/current.md
  archive_path_template: ~/hermes-snapshots/archive/constitution-<commit>-<date>.md
  index_path: ~/hermes-snapshots/current.index.json
  source_repo: ~/projects/hermes-constitution
```

`current.md` is the stable load target. It may be overwritten when the
constitution is reloaded.

`archive/constitution-<commit>-<date>.md` is an optional historical copy for
manual audit and rollback.

`current.index.json` records the snapshot blocks used to build `current.md`.
It is generated output and must not be stored in `hermes-constitution`.

## Default Behavior

Normal sessions should load the snapshot, not the full constitution.

```text
Default startup:
  /load-constitution-snapshot ~/hermes-snapshots/current.md
```

Hermes should cite the snapshot `constitution_version` in task outputs when
policy matters.

<!-- snapshot:block id="constitution-version-attribution" section="Constitution Snapshot" priority="30" -->
Task reports must attribute policy decisions to the loaded constitution
snapshot version. Hermes must read `constitution_version` from the loaded
snapshot or its generated index, not from the target project repository. A
missing `.hermes/constitution.md` file in a target project does not make the
constitution version `N/A`.
<!-- /snapshot:block -->

## Reload Conditions

Hermes should reload the full constitution only when:

- the user explicitly requests `reload constitution` or `/reload-constitution`
- the source repository git `HEAD` changed
- the snapshot is missing
- the snapshot version does not match the requested constitution version
- schema validation fails
- policy conflict is detected
- Hermes behavior conflicts with the loaded snapshot

## Commands

### `/load-constitution-snapshot [path]`

Loads a snapshot as the current session constitution cache.

Effects:

- read the snapshot file only
- do not read the full `~/projects/hermes-constitution` repository
- output `constitution_version`, `snapshot_path`, and `loaded_at`
- do not execute project tasks
- do not mutate project files
- do not write long-term memory

Default path:

```text
~/hermes-snapshots/current.md
```

### `/reload-constitution`

Reloads the full constitution and regenerates the snapshot.

Effects:

- read `~/projects/hermes-constitution`
- get current git `HEAD`
- scan declared snapshot blocks from source documents
- generate a Simplified Chinese snapshot from the block set
- overwrite `~/hermes-snapshots/current.md`
- archive a copy at `~/hermes-snapshots/archive/constitution-<commit>-<date>.md`
- write `~/hermes-snapshots/current.index.json`
- output added / changed / removed block ids compared with the previous index
- output the new `constitution_version`
- do not execute project tasks
- do not mutate target projects
- do not write long-term memory unless explicitly approved

## Source-Driven Snapshot Blocks

Snapshot generation must be source-driven. The snapshot generator must not rely
on one large hand-maintained template as the authority for policy content.

Source documents may declare required snapshot content with explicit block
markers:

```md
<!-- snapshot:block-example id="dependency-policy" section="Project Policy" priority="40" -->
Dependency changes are detected by the target project's dependency profile.
Dependency manifest and lockfile changes require approval by default.
<!-- /snapshot:block-example -->
```

Block rules:

- `id` must be stable and unique across the repository
- `section` groups related blocks in the generated snapshot
- `priority` controls ordering inside a section
- block body is policy content and must be preserved semantically
- the English source document remains the protocol source of truth
- Chinese source blocks may be used for operator-facing snapshot wording

`/reload-constitution` must build `current.md` by scanning source documents,
extracting snapshot blocks, sorting them by section and priority, and rendering
the resulting block set.

The previous snapshot is not a source of truth. The generator may compare with
the previous snapshot index to report changes, but it must not patch old
`current.md` as if that file were authoritative.

## Snapshot Index

Each reload should write an index next to the current snapshot:

```json
{
  "schema": "hermes.constitution_snapshot_index.v1",
  "constitution_version": "acc73eb",
  "generated_at": "2026-07-06T00:00:00Z",
  "source_repo": "~/projects/hermes-constitution",
  "snapshot_path": "~/hermes-snapshots/current.md",
  "blocks": [
    {
      "id": "dependency-policy",
      "section": "Project Policy",
      "priority": 40,
      "source": "docs/project-policy.md",
      "hash": "sha256:..."
    }
  ]
}
```

The index is used to detect and report:

- `added` blocks
- `changed` blocks
- `removed` blocks
- duplicate block ids
- missing required sections

Change detection must not decide policy by filename-only heuristics. It should
use block ids and content hashes.

## Snapshot Content and Format

A useful snapshot should include:

- constitution version / git commit
- execution plane policy
- provider roles
- provider auth policy
- model backends
- language policy
- dry-run policy
- memory policy
- human intervention policy
- simple shell direct mode
- risk and review policy
- provider adapter boundaries
- reload conditions

Snapshots should optimize for operator reviewability, not maximum compression.
The goal is to reduce repeated constitution loading while keeping policy easy to
audit by a human.

Allowed formatting:

- compact tables
- matrix-style blocks
- short grouped lists
- stable section numbering
- concise examples for command or execution rules

Prefer clearer table or matrix structure for:

- provider roles
- provider auth and surface separation
- model backends
- WSL / Windows execution plane
- language policy
- dry-run and memory policy
- human intervention budgets and stop conditions
- command handler effects and forbidden effects
- simple shell direct-mode allowlist

Snapshot length is a soft optimization, not a correctness limit. A snapshot
must not omit required policy merely to stay under a target line count.

Guidance:

- prefer concise wording when clarity is preserved
- keep all current critical policy sections represented
- preserve explicit budgets, stop conditions, allowlists, and forbidden actions
- preserve dependency profile rules for manifests and lockfiles
- preserve all declared snapshot blocks unless a block is invalid
- report omitted, duplicate, or malformed blocks as generation defects
- allow the snapshot to exceed 180 lines when the constitution grows
- treat missing policy as a defect, even if the snapshot is short

A shorter snapshot is not better if it loses information or becomes harder to
review, compare, or clean up manually.
