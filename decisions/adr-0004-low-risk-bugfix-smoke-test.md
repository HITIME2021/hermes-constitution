# ADR 0004: Low-Risk Bugfix Smoke Test

## Status

Accepted with reporting caveat.

## Date

2026-07-06.

## Context

After the initial test-only provider orchestration smoke test, Hermes needed a
higher-risk validation step that touched production code while staying outside
protected domains.

The chosen task was a low-risk CLI input normalization fix in
`GitHub-Trend-Intelligence-Platform`.

## Test

Target project:

```text
~/projects/GitHub-Trend-Intelligence-Platform
project_version: v0.2.3
project_commit: ed36453
```

Branch isolation:

```text
original_branch: main
test_branch: hermes-smoke/resolve-category-strip
```

Provider chain:

```text
Codex
  -> CodeBuddy
  -> pytest
  -> ad-hoc verification
```

Task:

```text
Normalize surrounding whitespace in main.py::_resolve_category.
```

Expected behavior:

```text
" ai"       -> "AI/ML"
"ai "       -> "AI/ML"
"\tweb\n"   -> "Web Framework"
"  xyz  "   -> "xyz"
"   "       -> ""
```

## Result

The low-risk bugfix smoke test passed.

Changed files:

```text
main.py            |  4 ++--
tests/test_main.py | 17 +++++++++++++++++
2 files changed, 19 insertions(+), 2 deletions(-)
```

Verification:

```text
python -m pytest tests/test_main.py -v -> 17 passed
ad-hoc verification script              -> 10/10 passed
```

Provider findings:

- Codex and CodeBuddy independently agreed the fix was correct.
- Both providers classified the regression risk as low.
- The change only affects whitespace-padded category input.
- Existing non-whitespace aliases keep their previous behavior.
- Unknown values are stripped before passthrough.
- Pure whitespace becomes an empty string.

Dependency profile check:

```text
requirements.txt: unchanged
requirements-dev.txt: unchanged
lockfile: none present
```

Stop conditions:

- Initial dirty worktree was detected and stopped.
- The operator chose stash-based cleanup before continuing.
- The test then ran on an isolated branch.
- No allowed-scope violation occurred.
- No dependency manifest or lockfile changed.
- No credential directory read occurred.
- No git commit or push was performed by the test.

## Reporting Caveat

The report listed:

```text
constitution_version: N/A (project has no .hermes/constitution.md)
```

This is a reporting defect. Constitution version attribution must come from the
loaded constitution snapshot or its index, not from the target project.

This ADR adds a constitution rule requiring Hermes task reports to cite the
loaded snapshot `constitution_version` and forbidding target-project-based
inference for this field.

## Decision

Hermes v0.1 is accepted as passing a low-risk production-code bugfix smoke
test, with the reporting caveat above.

The implementation change itself is recommended to keep, subject to operator
approval of this behavior:

```text
--cat "   " normalizes to an empty category string.
```

## Consequences

Positive:

- Hermes successfully handled a scoped production-code change.
- Branch isolation prevented accidental mutation of the main worktree.
- The provider chain caught and reported behavior-level edge cases.
- Dependency boundary checks used the project dependency profile.

Remaining limits:

- This is still not validation for auth, security, db, infra, deployment, or
  dependency-changing tasks.
- Medium-risk work still needs separate validation.
- Run observability remains limited because provider progress is visible mostly
  through final structured reports rather than a live run dashboard.
