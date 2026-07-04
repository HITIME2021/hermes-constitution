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
  source_repo: ~/projects/hermes-constitution
```

`current.md` is the stable load target. It may be overwritten when the
constitution is reloaded.

`archive/constitution-<commit>-<date>.md` is an optional historical copy for
manual audit and rollback.

## Default Behavior

Normal sessions should load the snapshot, not the full constitution.

```text
Default startup:
  /load-constitution-snapshot ~/hermes-snapshots/current.md
```

Hermes should cite the snapshot `constitution_version` in task outputs when
policy matters.

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
- generate a Simplified Chinese snapshot
- overwrite `~/hermes-snapshots/current.md`
- archive a copy at `~/hermes-snapshots/archive/constitution-<commit>-<date>.md`
- output the new `constitution_version`
- do not execute project tasks
- do not mutate target projects
- do not write long-term memory unless explicitly approved

## Snapshot Content and Format

A useful snapshot should include:

- constitution version / git commit
- execution plane policy
- provider roles
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
- model backends
- WSL / Windows execution plane
- language policy
- dry-run and memory policy
- human intervention budgets and stop conditions
- command handler effects and forbidden effects
- simple shell direct-mode allowlist

Default target size is 120-180 lines. A shorter snapshot is not better if it
becomes harder to review, compare, or clean up manually.
