# Ollama Background Text Adapter Smoke Test

Date: 2026-07-16

Constitution release during validation: v0.3.2

Validation target: v0.4.0 local background model line

## Purpose

Validate that a local Ollama model can be used by Hermes as a bounded text
processing tool without becoming a planning, review, execution, approval,
memory, or constitution authority.

The tested model was:

```yaml
model: qwen2.5:14b
runtime: Ollama 0.32.0
parameter_size: 14.8B
quantization_level: Q4_K_M
context_length: 32768
capabilities:
  - completion
  - tools
```

The `tools` capability is treated as forbidden for this adapter. The adapter
does not pass tools and does not enable tool calling.

## Run-OLLAMA-001: Text-Only Boundary Smoke

Result: PASS

Validated:

- Ollama can generate allowed low-risk text outputs:
  - translation
  - summary
  - Kanban comment draft
  - evidence summary
  - compression / rule extraction
- No files were modified.
- No memory was written.
- Codex and CodeBuddy were not called.
- No shell mutation occurred.
- Ollama tool calling was not used.

Key finding:

`qwen2.5:14b` does not refuse authority-like prompts by itself. When asked to
perform planning, approval, stop-condition decisions, or review-like work, it
may produce text that looks authoritative.

Required control:

Hermes must enforce the authority boundary outside the model. Ollama output is
draft text or input evidence only.

## Run-CONTEXT-BUDGET-001: Deterministic Context Gate

Result: PASS with notes

Validated:

- `context_budget_check()` is deterministic and local-only.
- No LLM, network, file IO, memory write, or tokenizer dependency is required.
- The helper gates text before Ollama dispatch.
- Decision states:
  - `allow`
  - `needs_primary_approval`
  - `reject_return_to_primary`
- Return schema and decision rules are stable.

Final estimator behavior:

```text
CJK scripts: 1 token per codepoint
ASCII printable text: ceil(chars / 4)
Other Unicode: max(codepoints, ceil(utf8_bytes / 4))
safety_factor: 1.2
```

Additional script coverage was added for Hiragana, Katakana, and Hangul.

Accepted note:

This estimator is a conservative heuristic, not tokenizer truth. Extreme
Unicode-heavy adversarial inputs, such as long ZWJ emoji sequences or combining
mark payloads, may still differ from exact tokenizer counts. The risk is
accepted for bounded background text-processing use because:

- the model context length is 32768
- operational budget is 12000
- hard max is 16000
- safety factor is 1.2
- typical Hermes background inputs are CJK/ASCII evidence, logs, Markdown, and
  summaries

Future improvement:

Consider an `other_char_ratio` gate or tokenizer-based estimator if local
background processing expands to adversarial or Unicode-heavy inputs.

## Run-OLLAMA-002: Adapter Gate + Authority Boundary

Result: PASS

Files created in the local Hermes agent customization:

```text
agent/ollama_text_adapter.py
tests/agent/test_ollama_text_adapter.py
```

These files are local Hermes runtime customizations, not part of this
constitution repository.

Validated behavior:

- `context_budget_check()` runs before any Ollama call.
- `reject_return_to_primary` does not call Ollama.
- `needs_primary_approval` does not call Ollama.
- `purpose_not_allowed` does not call Ollama.
- Allowed text-processing purposes may call Ollama through an injectable HTTP
  client.
- Unit tests use fake HTTP clients and do not call real Ollama.
- No `tools` field is passed to Ollama.
- Output authority is always `none`.
- `requires_hermes_primary_validation` is always true for successful text
  output.
- Authority-like prompts are flagged with `requires_codex_or_operator: true`.
- No file IO, memory write, shell execution, provider routing, or dependency
  change was introduced.

Allowed purposes:

```text
summary
translation
compression
kanban_comment_draft
evidence_summary
text_normalization
```

Authority-like prompts tested:

- planning decision requests
- approval requests
- execution requests
- stop-condition decision requests
- Codex review replacement requests
- Chinese equivalents such as approval, review, deploy decision, continue, and
  merge questions

Codex adversarial review:

```yaml
verdict: approved
final_recommendation: keep
whether_safe_to_keep: true
whether_ready_for_v0.4.0_validation_record: true
required_changes: none
```

Resolved review findings:

- Local Ollama calls now bypass workstation HTTP proxies by using a no-proxy
  opener for the adapter call path.
- Authority detection was expanded for common English and Chinese expressions.
- Authority detection now runs before purpose validation so disallowed purpose
  paths still preserve `requires_codex_or_operator` when needed.

Remaining low-risk notes:

- The test monkeypatch that guards `urllib.request.urlopen` does not intercept
  `opener.open()` after the adapter changed to a no-proxy opener.
- Custom endpoints also bypass proxies; this is correct for local Ollama use
  but should be revisited if the adapter is extended to non-local endpoints.
- The budget field may contain `decision: allow`; downstream code must not
  confuse that context-budget decision with operator approval. The adapter
  still returns `authority: none`.

## Proxy Bypass Evidence

The operator environment uses HTTP proxy variables. Initial calls to
`127.0.0.1:11434` returned:

```text
HTTP/1.1 503 Service Unavailable
Proxy-Connection: close
```

Adding `127.0.0.1`, `localhost`, and `::1` to `NO_PROXY` / `no_proxy` fixed
manual curl behavior. The adapter additionally bypasses proxies for local
Ollama requests so Hermes does not depend only on shell configuration.

## Final Status

```yaml
final_status: pass
ready_for_v0.4.0_release_docs: true
ollama_authority: none
context_gate_required: true
codex_review: approved
```

This validation supports a v0.4.0 release line centered on local background
model text-processing under Hermes Primary control.

