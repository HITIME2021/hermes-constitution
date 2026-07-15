# Gateway DM Session History Smoke Test

Date: 2026-07-16

Constitution release: v0.3.2

## Purpose

Record the Gateway / Weixin DM entry investigation where startup verification
correctly loaded the current constitution snapshot, but the DM response kept
returning an old constitution version.

This is a validation and incident record. It is not an ADR and does not change
the human-facing release label.

## Observed Problem

The Weixin DM entry returned stale constitution data:

```text
constitution_version: c2abe7d
release: v0.3.1
source_repo: ~/projects/production/hermes-constitution/
```

At the same time, gateway logs showed that the runtime guard loaded the current
snapshot:

```text
constitution guard loaded: version=877e5df release=v0.3.2 entry=weixin_dm
```

## Failed Hypotheses Tested

The following changes or checks did not resolve the stale response:

- Full snapshot injection into the gateway entry prompt.
- Fresh `sessions.json` mapping edit without resetting persisted history.
- Compact startup verification packet.
- Moving the startup packet ahead of the main system prompt.

These tests narrowed the problem away from snapshot path resolution and toward
restored DM conversation history.

## Root Cause

The stale `c2abe7d` value came from persisted Weixin DM conversation history in
Hermes `state.db`.

The active Weixin DM session contained prior assistant replies that stated
`c2abe7d` / `v0.3.1`. Each new DM turn reloaded those prior messages and sent
them back to the model as conversation history. The model repeated its earlier
assistant messages even though the gateway guard injected the current startup
verification packet.

Editing only `sessions.json` was insufficient because the gateway could
reattach the old persisted session from `state.db`.

## Recovery

The operator approved a DM `/new` session reset.

After `/new`, the next smoke test changed from the stale value to the current
guarded value:

```text
constitution_version: 877e5df
release: v0.3.2
source_repo: /home/hitime/projects/production/hermes-constitution
entry_channel: weixin_dm
```

The DM renderer truncated some field names and values in the displayed table.
That is a separate output-format issue, not the root cause of the stale
constitution version.

## Policy Lesson

For constitution-sensitive questions, gateway startup verification has higher
authority than restored conversation history.

When a gateway or DM session history contains stale assistant messages about
`constitution_version`, release, source repository path, or workspace layout,
Hermes must invalidate or reset that session before treating the channel as
trusted for constitution-sensitive work.

For manual recovery, `/new` is the preferred low-risk path. Direct database
editing should be reserved for cases where `/new` fails and should require a
separate dry-run with explicit operator approval.

For normal operation, Hermes should not rely on the operator to run `/new` after
each constitution snapshot change. Conversational gateway entrypoints should
bind sessions to the loaded `constitution_version` and automatically invalidate
or replace stale history before provider dispatch when the loaded version
changes.

## Follow-Up

- Keep Gateway Entry Guard logging and startup verification.
- Implement automatic DM session invalidation on `constitution_version`
  mismatch.
- Audit any diagnostic patches separately before deciding whether to keep or
  revert them.
- Track DM output truncation as a separate issue.
