# ADR 0011: Gateway Entry Guard and Self-Improvement Governance

Status: Accepted

Date: 2026-07-15

## Context

Hermes v0.3.1 made mobile and gateway workflows more important. The operator
expects to use Weixin / DM entrypoints for roughly 30 percent of future work.

A dry-run audit found that the Gateway / DM path creates agents through a
different code path than the normal TUI path:

- Gateway agents did not explicitly load `~/hermes-snapshots/current.md`.
- Gateway agents did not explicitly load `~/hermes-snapshots/current.index.json`.
- The observed DM response could report a stale `constitution_version` from
  restored session context or agent cache.
- Gateway restart triggered background self-improvement review, which patched a
  local `SKILL.md`.

Self-improvement is a useful Hermes growth mechanism, but gateway startup and
DM cold start are not appropriate places for silent skill mutation.

## Decision

Bump the human-facing constitution release from `v0.3.1` to `v0.3.2`.

Add Gateway Entry Guard:

- Gateway / DM / mobile / webhook entrypoints are untrusted until startup
  verification succeeds.
- They must load and verify `current.md` and `current.index.json` before tool
  execution.
- If verification fails, they must return `UNTRUSTED_CONTEXT_STOP`.
- They must not execute tools, code, shell commands, memory writes, provider
  work, or self-improvement before verification.
- Agent cache must be invalidated when the cached `constitution_version` differs
  from `current.index.json`.

Add Self-Improvement Governance:

- Automatic self-improvement analysis is allowed.
- Automatic candidate generation is allowed.
- Self-improvement writes are authority-bearing effects.
- Gateway startup and DM cold start may create candidates but must not apply
  patches.
- Applying a candidate requires trusted channel, scope, evidence, review, and
  approval.

## Consequences

- Weixin / DM can become a trusted work entrypoint only after startup guard is
  implemented and verified.
- Hermes keeps its learning mechanism without allowing silent background
  mutation in untrusted entrypoints.
- Future local Hermes runtime patches should implement candidate generation
  rather than direct gateway startup patching.

## Rejected Alternatives

- Disable self-improvement entirely.
  - Rejected because Hermes should keep learning.
- Allow gateway startup to patch skills automatically.
  - Rejected because service startup is not an approved execution channel.
- Treat DM entrypoints as equivalent to TUI sessions.
  - Rejected because the audited code path does not load the constitution
    snapshot by default.
