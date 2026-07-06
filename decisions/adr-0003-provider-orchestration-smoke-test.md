# ADR 0003: Provider Orchestration Smoke Test

## Status

Accepted.

## Date

2026-07-06.

## Context

Hermes v0.1 defines Codex as the planning and review provider, CodeBuddy as the
scoped execution provider, and DeepSeek-V4-Pro as a model backend only.

After introducing source-driven constitution snapshots, Hermes needed a live
smoke test to verify that the provider orchestration path follows the
constitution instead of calling a single provider directly.

## Test

Constitution snapshot:

```text
constitution_version: 87a7b3e
snapshot: ~/hermes-snapshots/current.md
index: ~/hermes-snapshots/current.index.json
```

Target project:

```text
~/projects/GitHub-Trend-Intelligence-Platform
```

Task:

```text
Add boundary tests for main.py::_resolve_category.
```

Required provider chain:

```text
Codex planning
  -> CodeBuddy scoped execution
  -> pytest verification
  -> Codex review
```

Provider readiness:

| Provider | CLI path | Version | Status |
|----------|----------|---------|--------|
| Codex | `/home/hitime/.local/bin/codex` | `0.142.5` | Ready |
| CodeBuddy | `/home/hitime/.local/bin/codebuddy` | `2.117.0` | Ready |
| DeepSeek | n/a | n/a | Not used |

## Result

The live smoke test passed.

Observed result:

- Codex performed planning.
- Codex planning used read-only sandboxing and produced a Task,
  ExecutionRequest, and ReviewPlan.
- CodeBuddy performed scoped execution.
- `pytest` verification passed.
- Codex performed final review.
- DeepSeek did not participate in planning or review.
- No long-term memory writeback was performed.
- No git commit or push was performed by the test.
- All declared human intervention stop conditions remained untriggered.

Execution summary:

```text
files_changed: tests/test_main.py
lines_added: 21
lines_removed: 0
```

Verification summary:

```text
pytest tests/test_main.py::TestResolveCategory -v -> 9 passed
pytest tests/ -q                                  -> 244 passed
```

Codex review result:

```text
verdict: approved
scope_boundary: pass
no_dependency_change: pass
no_production_code_change: pass
test_preservation: pass
no_private_alias_import: pass
semantic_correctness: pass
```

Dependency policy was evaluated through the target project's dependency
profile. For this project, Hermes identified a Python pip-based dependency
profile with:

```text
requirements.txt
requirements-dev.txt
```

Hermes did not hardcode `pyproject.toml` as the dependency boundary.

Human intervention stop conditions:

```text
dependency manifest or lockfile change: not triggered
allowed_scope violation: not triggered
repeated test failure: not triggered
provider credential directory read: not triggered
provider auth failure: not triggered
scope expansion: not triggered
Codex planning failure: not triggered
Codex review scope violation: not triggered
```

## Decision

The v0.1 provider orchestration pipeline is accepted as passing its initial
live smoke test.

This validates the basic separation of responsibilities:

```text
Codex = planning and review
CodeBuddy = scoped execution
Hermes = policy, routing, verification, and stop-condition control
DeepSeek = model backend only, unless Project Policy explicitly changes it
```

## Consequences

Positive:

- The v0.1 provider role model has been validated on a low-risk live task.
- The source-driven constitution snapshot was sufficient for provider routing
  and dependency policy interpretation.
- Dependency change detection used the project dependency profile rather than a
  hardcoded manifest filename.

Remaining limits:

- This was a low-risk smoke test, not broad production certification.
- Medium-risk tasks still require separate validation with Codex planning and
  review.
- Protected domains such as auth, security, db, infra, deployment, and
  dependency changes still require explicit approval and stricter review.
