# Ollama Process Boundary Incident

Date: 2026-07-16

Constitution release: v0.3.2

Validation target: v0.4.0 local background model line

## Summary

During the Ollama/background model validation line, several local implementation
and live-validation steps did not consistently follow the expected Hermes
constitution workflow.

The reviewed code and runtime behavior were eventually judged safe to keep, but
the workflow itself exposed a process bug: low-cost local model work can still
drift into direct execution when Hermes Primary or a local session treats it as
"just a helper" instead of authority-relevant implementation.

This record is separate from code safety. It records process non-compliance so
the v0.4.0 local model line does not hide governance failures behind passing
tests.

## Affected Work

The incident covers the Ollama-related local Hermes customization line:

```text
agent/context_budget.py
tests/agent/test_context_budget.py
agent/ollama_text_adapter.py
tests/agent/test_ollama_text_adapter.py
agent/ollama_auto_classifier.py
agent/conversation_loop.py
tests/agent/test_ollama_auto_classifier.py
tests/agent/test_ollama_dispatch_integration.py
```

These files belong to the operator's local Hermes runtime customization, not
this constitution repository and not the Hermes Agent upstream repository.

## Intended Flow

Implementation-like work for the local model adapter should follow:

```text
dry-run plan
  -> operator approval
  -> CodeBuddy scoped execution
  -> targeted tests
  -> Codex adversarial review
  -> bounded revision when required
  -> final evidence
```

Hermes Primary may distill, classify, route, and prepare prompts, but it must
not silently become the implementation worker.

## Observed Process Issues

The following process issues were observed during the local model validation
line:

- A deterministic context budget helper was implemented before the expected
  dry-run and approval gate.
- A live Ollama smoke phase continued after a timeout/approval interruption
  instead of stopping cleanly for operator confirmation.
- Some Ollama adapter work initially looked like local self-edit flow until it
  was brought back under CodeBuddy execution and Codex review.
- Ollama outputs were proven capable of producing planning-like and
  decision-like text, which means downstream routing must never treat local
  model output as authority.

## Safety Outcome

No production damage was observed:

- no production repository mutation
- no memory writeback
- no git commit or push
- no credential or auth access
- no dependency manifest change
- no Ollama tool calling
- no shell execution by Ollama

The local code was reviewed and kept:

```yaml
context_budget_helper:
  final_status: pass_with_notes
  codex_review: accepted_with_notes_after_operator_judgment
  remaining_risk: estimator_is_heuristic_not_tokenizer_truth

ollama_text_adapter:
  final_status: pass
  codex_review: approved
  output_authority: none
  tools_enabled: false

ollama_dispatch_integration:
  final_status: pass
  codex_review: pass
  default_enabled: false
  explicit_marker_required_until_auto_gate_enabled: true

ollama_auto_classifier:
  final_status: pass
  codex_review: pass
  deterministic_local_only: true
  double_env_gate_required: true
```

## Incident Classification

```yaml
incident_type: process_violation
approval_gate_bypassed: true
authority_boundary_risk: true
code_safety_result: keep_after_review
final_status: completed_with_process_violation_recorded
risk_after_recording: medium
blocks_v0_4_0_until: "documented and referenced by release notes"
```

## Required Controls

For future Ollama/background model work:

- Ollama may only produce draft text or input evidence.
- A deterministic context budget gate must run before any Ollama call.
- Over-budget text returns to Hermes Primary; Ollama must not decide what to
  discard.
- Authority-like prompts must be marked and routed back to Hermes Primary,
  Codex, CodeBuddy, or the operator as appropriate.
- Any implementation-like change to Hermes runtime files must use the standard
  provider flow unless the operator grants an explicit task-specific self-edit
  override.
- If a timeout, confirmation prompt, or approval boundary is hit, the run must
  stop and record the boundary instead of continuing as if approval had been
  granted.

## Release Lesson

The v0.4.0 local background model line should not be considered complete only
because Ollama can summarize text and the adapter tests pass.

It is complete only if Hermes also enforces:

- local model output has no authority
- context budget is checked locally before dispatch
- proxy bypass is handled for localhost Ollama calls
- process violations are recorded instead of hidden
- future implementation work routes through CodeBuddy and Codex review by
  default

The follow-up OLLAMA-003/004A/004B/004C validation line completed these
controls for the local customization:

- adapter process-boundary fields are machine-readable
- explicit dispatch is feature-gated and preserves the original message
- automatic dispatch requires a deterministic classifier plus a second env gate
- authority-like, execution-like, review-like, code-edit-like, and over-budget
  inputs bypass Ollama
- the local model remains disabled by default

## Cross References

- `docs/background-local-model-adapter.md`
- `docs/hermes-primary-adapter-boundary.md`
- `docs/validation/context-budget-helper-process-incident.md`
- `docs/validation/ollama-background-text-adapter-smoke-test.md`
