# Context Budget Helper Process Incident

Date: 2026-07-16

Constitution release: v0.3.2

## Summary

During Ollama/background model preparation, a deterministic context budget
helper was implemented before the expected dry-run and operator approval gate.

The implementation itself was reviewed and judged safe to keep, but the workflow
violated the intended Hermes Primary boundary.

## Files Created

```text
agent/context_budget.py
tests/agent/test_context_budget.py
```

These files belong to the local Hermes agent customization, not this
constitution repository.

## Intended Flow

```text
dry-run plan -> operator approval -> CodeBuddy scoped execution
  -> tests -> Codex adversarial review
```

## Observed Flow

```text
implementation completed -> tests passed -> review after the fact
```

The expected dry-run and approval gate was bypassed.

## Review Outcome

Codex adversarial review judged:

- implementation correctness: PASS with non-blocking notes
- test sufficiency: PARTIAL
- keep vs rollback: KEEP
- process violation severity: MEDIUM

No damage was observed:

- pure Python stdlib helper
- no network
- no file IO
- no auth, database, memory, provider, config, gateway, or shell integration
- no commit or push

## Decision

Keep the helper, but record the process incident.

```yaml
approval_gate_bypassed: true
final_status: completed_with_process_violation_recorded
decision: keep
follow_up:
  - Hermes Primary Adapter Boundary
  - pre-execution self-edit guard
  - approval_gate_bypassed evidence field
  - optional Unicode and threshold test enhancement
```

## Policy Lesson

Hermes Primary must not infer self-edit approval from low task risk, small file
count, local-only scope, test-only scope, or implementation convenience.

Small deterministic helper work still needs dry-run planning, explicit
operator approval, scoped execution, verification, and review unless the
operator grants a task-specific Hermes self-edit override.

