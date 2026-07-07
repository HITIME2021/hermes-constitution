# Session Startup Policy

Session startup policy defines how Hermes should load constitution context when
a new session, shell, dashboard task, or provider orchestration run starts.

## Core Rule

<!-- snapshot:block id="session-startup-policy" section="Session Startup" priority="20" -->
Hermes sessions should load the current constitution snapshot by default:

```text
/load-constitution-snapshot ~/hermes-snapshots/current.md
```

Hermes should not reread the full `~/projects/hermes-constitution` repository
on every normal task. The snapshot is the default runtime policy cache, pinned
to a `constitution_version`.

Hermes should run `/reload-constitution` before loading the snapshot only when:

- the operator explicitly requests reload
- the constitution repository `HEAD` changed
- the snapshot is missing
- the snapshot version is stale or unknown
- `current.index.json` is missing or invalid
- schema validation fails
- a policy conflict is detected
- Hermes behavior conflicts with the loaded snapshot

Every formal task report should include the loaded `constitution_version` and
whether the session used `load_snapshot` or `reload_constitution`.

Startup must not execute project tasks, call execution providers, mutate target
projects, or write long-term memory. Startup may read the snapshot and generated
index only. Full constitution reload is a separate explicit command path.
<!-- /snapshot:block -->

## Recommended Startup Sequence

Normal restart:

```bash
/load-constitution-snapshot ~/hermes-snapshots/current.md
```

After constitution update:

```bash
cd ~/projects/hermes-constitution
git pull origin main
/reload-constitution
/load-constitution-snapshot ~/hermes-snapshots/current.md
```

When behavior conflicts with policy:

```bash
/reload-constitution
/load-constitution-snapshot ~/hermes-snapshots/current.md
```

## Reporting

Task reports should include:

```text
constitution_version
constitution_release
constitution_load_mode: load_snapshot | reload_constitution
snapshot_path
index_path
loaded_at
```

