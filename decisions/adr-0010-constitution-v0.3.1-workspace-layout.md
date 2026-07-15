# ADR 0010: Constitution v0.3.1 Workspace Layout Policy

Status: Accepted

Date: 2026-07-15

## Context

Hermes v0.3 added Tools Layer and Tools Adapter governance. The next practical
issue appeared during `spec-kit/specify` validation: smoke projects, lab
projects, production repositories, and provider worktrees were all being placed
directly under `~/projects`.

That layout makes it harder for Hermes and the operator to distinguish a real
production repository from a disposable tool experiment. It also increases the
risk that a frontend tool initializer or scaffold command is run in the wrong
directory.

## Decision

Bump the human-facing constitution release from `v0.3` to `v0.3.1`.

Add Workspace Layout Policy:

```text
~/projects/production/  -> real long-lived repositories
~/projects/labs/        -> smoke tests, tool validation, disposable experiments
~/projects/worktrees/   -> git worktrees and Hermes run workspaces
~/projects/archive/     -> obsolete experiments retained for inspection
```

Hermes must not create smoke, lab, or tool-validation projects directly under
`~/projects`. Tool experiments such as `spec-kit` smoke tests should run under
`~/projects/labs`.

Provider orchestration worktrees should use `~/projects/worktrees` unless the
operator explicitly chooses another path.

## Consequences

- Cleaner separation between production repositories and experiments.
- Lower risk of accidental scaffold or workflow mutation in production repos.
- Run evidence should include `workspace.class`.
- Existing repositories do not need to be moved immediately; this is the
  default for new work.

## Rejected Alternatives

- Keep using `~/projects` directly for everything.
  - Rejected because it blurs production and lab boundaries.
- Treat all directories under `~/projects` as production.
  - Rejected because smoke and tool-validation projects are intentionally
    disposable.
- Make the layout tool-specific.
  - Rejected because the policy should apply to all future frontend/backend
    tools, not just `spec-kit/specify`.
