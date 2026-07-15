# Gateway Entry Guard

Gateway Entry Guard defines the minimum trust checks required before Hermes may
handle work received through gateway, direct message, mobile, webhook, or other
non-TUI entrypoints.

## Core Rule

<!-- snapshot:block id="gateway-entry-guard" section="Gateway Entry Guard" priority="59" -->
Gateway, DM, mobile, webhook, and other non-TUI entrypoints are untrusted until
startup verification succeeds.

Before any tool call, code execution, shell command, memory writeback,
self-improvement action, or task execution, the entrypoint must load and verify:

```text
~/hermes-snapshots/current.md
~/hermes-snapshots/current.index.json
```

The entrypoint must produce a startup verification packet:

```yaml
startup_verification:
  entry_channel: weixin_dm
  snapshot_path: ~/hermes-snapshots/current.md
  index_path: ~/hermes-snapshots/current.index.json
  constitution_version: <commit>
  release: <human-facing release>
  source_repo: ~/projects/production/hermes-constitution
  workspace_layout:
    production: ~/projects/production/
    labs: ~/projects/labs/
    worktrees: ~/projects/worktrees/
    archive: ~/projects/archive/
```

If any required field cannot be verified, Hermes must stop with:

```text
UNTRUSTED_CONTEXT_STOP
```

Until startup verification succeeds, the entrypoint must not:

- call `execute_code`
- execute heredoc code
- execute shell commands
- call search/read/write tools
- write memory
- patch skills
- run self-improvement
- dispatch provider work
- process live ExecutionRequests

Gateway and DM channels may be used for notifications, pairing, approval
questions, and read-only status after startup verification. They must not rely
on stale session transcripts, LRU agent cache, restored conversation state, or
message history as constitution authority.

When a gateway or DM session contains prior assistant messages that mention an
older `constitution_version`, release, source repository path, or workspace
layout, those messages must be treated as stale evidence. Startup verification
has higher authority than restored conversation history.

If the loaded `constitution_version` or release changes for a DM user, Hermes
must invalidate or reset the affected DM session before using that channel for
constitution-sensitive work. A user-approved `/new` session reset is an
acceptable manual recovery path. Editing only a session mapping file is not
sufficient if persisted conversation rows can reattach the old session.

For conversational gateway entrypoints, the restored session must be bound to
the `constitution_version` loaded from `current.index.json`. If the restored
session version differs from the loaded version, Hermes must invalidate the old
conversation history before any provider call and create a fresh session for
that channel. Old assistant messages that mention prior constitution versions,
releases, source repository paths, or workspace layouts must not be sent to the
model as conversation history after such a mismatch.

Manual `/new` remains an acceptable recovery path, but normal operation must not
depend on the operator manually resetting a DM session after every constitution
snapshot change.

If the gateway process uses an agent cache, the cache must be invalidated when
`current.index.json` changes or when the cached `constitution_version` differs
from the current index.
<!-- /snapshot:block -->

## Rationale

The TUI path may load the constitution because the agent follows Session Startup
Policy. Gateway entrypoints are service processes and may create agents through
different code paths. They need an explicit guard so mobile or DM work uses the
same constitution snapshot as TUI work.

## Non-Goals

This policy does not require gateway channels to execute production work. A
deployment may intentionally restrict gateway channels to notification and
approval only.

This policy does not bypass user pairing, allowlists, platform auth, or
operator approval.
