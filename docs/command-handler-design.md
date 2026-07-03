# Command Handler Design

Command handlers turn user slash commands into safe, auditable control-plane
operations.

Commands are not workflow tasks. They do not perform provider execution and do
not directly mutate task state. They are session/control operations with
declared effects, explicit forbidden effects, and structured outputs.

## Design Principle

```text
command -> command policy gate -> handler -> effects -> result
```

A command handler:

- declares what it may do through `effects`
- declares what it must not do through `no_effects`
- receives structured input
- returns structured output
- never silently skips required declared effects
- never performs undeclared side effects
- never bypasses Project Policy, approval requirements, or filesystem scope

Commands do not go through the normal Workflow Engine, Task Planner, Risk
Evaluator, or Review Gate path for project tasks. They still pass a Command
Policy Gate before execution.

## Command Policy Gate

The Command Policy Gate checks a command before dispatch:

- the command alias resolves to one registered handler
- parameters match the command schema
- requested effects are allowed for the command
- forbidden effects are not requested by caller context
- required approvals are present
- file writes are inside the command's declared scope
- target project mutation is blocked unless explicitly declared and approved
- long-term memory writeback is blocked unless explicitly declared and approved

If the gate cannot prove the command is safe, it returns a structured command
error instead of dispatching the handler.

## Command Schema

```yaml
commands:
  <command_name>:
    aliases:
      - /<slash-alias>
    description: "<human-readable description>"
    owner: hermes_core
    parameters:
      <param_name>:
        type: <string|path|boolean|choice>
        default: <default_value>
        required: <true|false>
    effects:
      - <allowed_effect>
    no_effects:
      - <forbidden_effect>
    output:
      required_fields:
        - <field_name>
    errors:
      - <error_type>: <human-readable_message>
```

## Effect Catalog

`effects` describe what a command may do.

`no_effects` describe what a command must not do even if the surrounding
conversation suggests it.

| Effect | Meaning |
|--------|---------|
| `read_snapshot_only` | Read a snapshot file without touching the source repo |
| `read_full_constitution` | Read constitution docs, schemas, and decisions |
| `generate_snapshot` | Generate a new constitution snapshot from source docs |
| `overwrite_current_snapshot` | Overwrite `~/hermes-snapshots/current.md` |
| `archive_versioned_snapshot` | Write an archive copy under `~/hermes-snapshots/archive/` |
| `read_file` | Read one or more files |
| `write_file` | Write or overwrite one declared file |
| `archive_file` | Move or copy one declared file to an archive path |
| `require_approval` | Prompt the operator for approval before acting |

| No Effect | Meaning |
|-----------|---------|
| `full_constitution_reload` | Must not walk or reread the full constitution repository |
| `task_execution` | Must not invoke Workflow Engine task execution |
| `provider_execution` | Must not invoke Codex, CodeBuddy, or any provider adapter for execution |
| `project_mutation` | Must not mutate any project repository |
| `target_project_mutation` | Must not mutate the current target project |
| `source_repo_mutation` | Must not mutate the constitution source repository |
| `memory_writeback` | Must not write to long-term Memory Center |
| `provider_routing` | Must not route work to a provider |
| `task_state_transition` | Must not change Task state |
| `delete_file` | Must not delete files |

## Command Registry

The registry is a flat namespace owned by Hermes Core.

```text
CommandRegistry
  -> register(name, handler_metadata, handler)
  -> resolve(alias) -> handler
  -> list() -> [command_metadata]
```

Only Hermes Core may register built-in commands by default.

Skill-provided commands require explicit registration through Command Policy and
must declare their owning skill in metadata.

Provider adapters must not register user-facing slash commands by default.
Provider adapter commands are allowed only when Project Policy explicitly
authorizes them and the command does not bypass adapter scope, review, or
approval boundaries.

## Constitution Commands (v0.1)

### `load_constitution_snapshot`

Loads the pinned constitution snapshot for the current session.

```yaml
commands:
  load_constitution_snapshot:
    aliases:
      - /load-constitution-snapshot
    description: "Load the pinned constitution snapshot into the current session."
    owner: hermes_core
    parameters:
      path:
        type: path
        default: ~/hermes-snapshots/current.md
        required: false
    effects:
      - read_snapshot_only
    no_effects:
      - full_constitution_reload
      - task_execution
      - provider_execution
      - project_mutation
      - memory_writeback
      - provider_routing
      - task_state_transition
    output:
      required_fields:
        - constitution_version
        - snapshot_path
        - loaded_at
    errors:
      - snapshot_not_found: "Snapshot file does not exist. Run /reload-constitution first."
      - snapshot_version_mismatch: "Snapshot version does not match requested version."
      - permission_denied: "Cannot read the snapshot file."
```

The handler must not read `~/projects/hermes-constitution` unless the snapshot
is missing or stale and the user explicitly authorizes `/reload-constitution`.

### `reload_constitution`

Reloads the full constitution from the source repository and regenerates the
snapshot.

```yaml
commands:
  reload_constitution:
    aliases:
      - /reload-constitution
    description: "Reload the full constitution and regenerate the current snapshot."
    owner: hermes_core
    parameters:
      source_repo:
        type: path
        default: ~/projects/hermes-constitution
        required: false
      current_snapshot:
        type: path
        default: ~/hermes-snapshots/current.md
        required: false
      archive_dir:
        type: path
        default: ~/hermes-snapshots/archive
        required: false
    effects:
      - read_full_constitution
      - generate_snapshot
      - overwrite_current_snapshot
      - archive_versioned_snapshot
    no_effects:
      - task_execution
      - provider_execution
      - target_project_mutation
      - source_repo_mutation
      - memory_writeback
      - provider_routing
      - task_state_transition
      - delete_file
    output:
      required_fields:
        - constitution_version
        - commit
        - snapshot_path
        - archive_path
        - loaded_at
    errors:
      - source_repo_not_found: "Constitution repository not found at the expected path."
      - git_head_unavailable: "Cannot determine current git HEAD."
      - snapshot_write_failed: "Cannot write the generated snapshot."
      - archive_write_failed: "Cannot write the archived snapshot."
```

The handler may write snapshot files under `~/hermes-snapshots/`. It must not
write generated snapshots into `~/projects/hermes-constitution`.

## Caller Contract

Callers that invoke a command handler must:

- pass parameters exactly as declared
- respect `effects` and `no_effects`
- handle declared error types
- not bypass the handler to call internal functions directly
- not layer task execution, provider routing, memory writeback, or file mutation
  onto a command whose `no_effects` forbid them

A gateway, CLI, or chat surface that wraps commands must not silently add
behavior outside the command contract.

## Testing

Command handlers should be tested in isolation.

The test contract:

- every declared effect is verifiable
- every declared error is triggerable
- every `no_effect` is enforced
- output schema matches declared `required_fields`
- command policy gate rejects undeclared or conflicting effects
- command execution does not mutate Task state unless explicitly declared

