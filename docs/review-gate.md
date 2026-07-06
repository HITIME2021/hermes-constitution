# Review Gate

Review Gate decides whether an execution result can move forward, needs
revision, must be escalated, is blocked, or has failed.

## Review Levels

```text
L0 No Review
L1 Boundary Review
L2 Automated Verification
L3 Codex Review
L4 Human Approval
```

## L1 Boundary Review

Checks:

- only allowed files were modified
- no forbidden files were touched
- no unapproved dependency was added
- no protected path was modified
- no secrets appeared
- no destructive action occurred

## L2 Automated Verification

Checks may include:

- lint
- typecheck
- unit tests
- build
- targeted tests
- static scan

## L3 Codex Review

Codex checks:

- requirement fit
- scope control
- design quality
- correctness
- edge cases
- test adequacy
- security, data, performance, and compatibility risk
- maintainability
- memory candidates

<!-- snapshot:block id="codex-review-modes" section="Review Gate" priority="70" -->
## Codex Review Modes

Codex review has two modes:

```text
normal       = standard correctness, scope, test, and maintainability review
adversarial  = stronger skeptical review that actively looks for hidden
               regressions, missing edge cases, unsafe scope expansion,
               security issues, dependency risk, and approval bypasses
```

`adversarial` review is required by default for:

- medium and higher risk tasks
- architecture, provider routing, Review Gate, memory, or constitution changes
- dependency manifest or lockfile changes
- auth, secrets, permissions, database, infrastructure, deployment, billing, or
  security-sensitive changes
- repeated test failures, repeated review findings, or replanning after a
  failed attempt
- any task where the operator explicitly asks for stronger review

Codex review is review-only. During a review phase, the reviewer must not edit
files, run scoped execution as an executor, approve its own unreviewed changes,
or silently expand the task scope. Required fixes must become a new
ExecutionRequest or a bounded revision phase before they are executed.

Review comments should distinguish:

```text
approved
approved_with_notes
needs_revision
blocked
escalate_to_human
```

If adversarial review finds a blocking issue, the task must not enter
completed state until the issue is resolved, explicitly waived by the operator,
or escalated under Human Intervention Policy.
<!-- /snapshot:block -->

## L4 Human Approval

Required for:

- critical risk
- production deployment
- database migration
- destructive operation
- secrets or permission changes
- payment or billing logic
- irreversible changes

Also required when Human Intervention Policy stop conditions require operator
judgment, such as repeated failed revisions, repeated review findings, unsafe
scope expansion, or unclear strategic direction.

## Risk to Review Mapping

```text
trivial -> L0 or L1
low -> L1 + optional L2
medium -> L1 + L2 + L3
high -> L1 + L2 + L3 + optional L4
critical -> L1 + L2 + L3 + L4
```

## Decisions

```text
passed
needs_revision
blocked
escalate
failed
```

## Hard Rules

- CodeBuddy code changes require at least L1.
- Medium and higher risk tasks require L3 Codex review.
- Medium and higher risk tasks use `adversarial` Codex review unless Project
  Policy explicitly chooses a stronger human approval path instead.
- Codex review is read-only; the reviewer does not become the executor.
- High and critical work must include recovery or rollback consideration.
- Forbidden file modification escalates immediately.
- Secrets exposure blocks immediately.
- Failed tests cannot enter completed state.
- Review Gate can raise risk but cannot lower risk.
- The same required action repeated across two reviews must block automatic revision and request human intervention.
