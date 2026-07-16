# ADR 0012: Constitution v0.4.0 Local Background Model Governance

Status: Accepted

Date: 2026-07-16

## Context

Hermes v0.3.2 established trusted gateway entrypoints, self-improvement
governance, and a stricter Hermes Primary boundary. The next practical cost
problem was that many low-risk text-heavy operations still consumed the primary
LLM backend even though they did not require planning, review, execution, or
approval authority.

The operator installed and validated a local Ollama runtime with
`qwen2.5:14b`. The model can perform useful text processing, but it does not
reliably refuse authority-like requests by itself. When prompted, it may produce
planning-like, approval-like, or review-like text.

This means local models can reduce cost only if Hermes keeps all authority
outside the model.

## Decision

Bump the human-facing constitution release from `v0.3.2` to `v0.4.0`.

Add the v0.4.0 local background model line:

- Ollama/local models may be used for bounded text preprocessing.
- Ollama output is draft text or input evidence only.
- Ollama output has `output_authority: none`.
- A deterministic context budget gate must run before any Ollama call.
- Over-budget, authority-like, execution-like, review-like, code-edit-like, or
  scope-expansion-like inputs must bypass Ollama.
- The adapter must not pass `tools` or enable tool calling.
- Local Ollama traffic must bypass workstation HTTP proxies.
- Hermes Primary, Codex, CodeBuddy, and the operator remain the authority
  surfaces for planning, review, execution, approval, memory, and scope control.

Validate this with the OLLAMA run line:

```text
OLLAMA-003  -> adapter process-boundary enforcement
OLLAMA-004A -> adapter live cost smoke
OLLAMA-004B -> explicit-purpose dispatch hook
OLLAMA-004C -> deterministic auto classifier
```

## Consequences

- Hermes may reduce primary backend token use for low-risk text preprocessing.
- The local model remains disabled by default.
- Explicit dispatch requires `HERMES_OLLAMA_BACKGROUND_TEXT=1`.
- Automatic dispatch additionally requires
  `HERMES_OLLAMA_BACKGROUND_TEXT_AUTO=1`.
- Automatic dispatch is limited to high-confidence text preprocessing:
  `evidence_summary`, `translation`, `compression`, and
  `text_normalization`.
- Broader tasks such as generic planning, review, approval, execution,
  stop-condition decisions, memory writeback, scope expansion, dependency
  changes, auth, git, database, and infrastructure work bypass Ollama.
- Local runtime `.py` patches are documented as reference patch groups, not
  upstream Hermes Agent source.

## Validation Evidence

- `docs/validation/ollama-background-text-adapter-smoke-test.md`
- `docs/validation/ollama-process-boundary-incident.md`
- `docs/local-hermes-runtime-patches.md`

Validation summary:

```yaml
OLLAMA-003: pass
OLLAMA-004A: pass
OLLAMA-004B: pass
OLLAMA-004C: pass
risk_after_review: low
total_tests_reported: 143
codex_rounds_total: 6
estimated_deepseek_tokens_saved_in_004a: 4700
default_state: disabled
ollama_authority: none
```

## Rejected Alternatives

- Use Ollama as a planning or review substitute.
  - Rejected because local model output can look authoritative and cannot
    replace Codex review or operator approval.
- Automatically route all long prompts to Ollama.
  - Rejected because long prompts often contain instructions, code changes,
    approvals, or provider-routing decisions.
- Require manual markers forever.
  - Rejected because it would not materially reduce primary backend cost in
    normal use.
- Enable automatic classification with an LLM.
  - Rejected because deciding whether to call Ollama must itself be local,
    deterministic, and auditable.
