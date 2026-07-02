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

## L4 Human Approval

Required for:

- critical risk
- production deployment
- database migration
- destructive operation
- secrets or permission changes
- payment or billing logic
- irreversible changes

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
- High and critical work must include recovery or rollback consideration.
- Forbidden file modification escalates immediately.
- Secrets exposure blocks immediately.
- Failed tests cannot enter completed state.
- Review Gate can raise risk but cannot lower risk.

