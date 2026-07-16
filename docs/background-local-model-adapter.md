# Background Local Model Adapter

Background Local Model Adapter defines how Hermes may use local Ollama models
such as `qwen2.5:14b` for bounded, low-risk text processing.

The adapter is intentionally weaker than a provider adapter. A local background
model may transform text, but it must not create authority-bearing effects.

## Core Rule

<!-- snapshot:block id="background-local-model-adapter" section="Tools Layer" priority="63" -->
Ollama background models are text-processing tools, not planning, review,
execution, approval, or memory authorities.

Allowed uses:

- prompt distillation
- evidence summary
- Kanban comment drafts
- final report drafts
- log grouping
- task text structuring
- translation
- duplicate removal
- token-noise reduction
- artifact text pre-processing
- candidate extraction for stop-condition checklists

Forbidden uses:

- final planning source of record
- Codex review replacement
- CodeBuddy execution replacement
- code, config, memory, database, auth, or credential mutation
- shell execution
- provider routing authority
- approval decisions
- final stop-condition decisions
- long-term memory writeback
- git commit, push, or pull request creation
- constitution authority
- deciding which evidence may be discarded when context is too large

Hermes must validate and route all Ollama output before it affects a task.
Ollama output is draft text or input evidence only.

## Context Budget Gate

Local model context is finite. For `qwen2.5:14b`, the observed Ollama model
metadata is:

```yaml
model: qwen2.5:14b
parameter_size: 14.8B
quantization_level: Q4_K_M
context_length: 32768
```

Hermes should use a conservative operating budget:

```yaml
ollama_background:
  model: qwen2.5:14b
  model_context_length: 32768
  operational_context_budget_tokens: 12000
  hard_max_input_tokens: 16000
```

Before dispatching text to Ollama, Hermes must estimate packet size. If the
packet may exceed the configured local model context budget, Hermes must not:

- chunk blindly
- recursively summarize without supervision
- silently drop content
- ask Ollama which evidence can be discarded

Instead, Hermes must route the task back to Hermes Primary for trimming,
segmentation, stronger provider selection, or operator judgment.

Context budget estimation must be deterministic local computation. Hermes must
not spend LLM tokens to decide whether text fits a local model context window.
If an exact tokenizer is unavailable, Hermes should use a conservative local
estimator and record the estimate method in evidence.

Routing rule:

```text
estimated_input_tokens <= 12000:
  Ollama may process the bounded text packet.

12000 < estimated_input_tokens <= 16000:
  Hermes Primary may approve bounded use only for low-risk text processing.

estimated_input_tokens > 16000:
  Do not call Ollama. Route back to Hermes Primary.
```

Ollama must never be the authority deciding what context can be omitted from
evidence, planning packets, or review packets.
<!-- /snapshot:block -->

## Adapter Boundary

The adapter may call a local Ollama API endpoint such as:

```text
http://127.0.0.1:11434/api/generate
```

The call is still a Backend Tool invocation because it consumes runtime
resources and may depend on local service state, but its permitted effect is
limited to text transformation. It must not receive credentials or raw private
material when a redacted summary is sufficient.

## Local Proxy Bypass

Many operator workstations run HTTP proxies for external network access. Local
Ollama traffic must not be routed through those proxies.

Common symptom:

```text
curl http://127.0.0.1:11434/api/tags
HTTP/1.1 503 Service Unavailable
Proxy-Connection: close
```

If the same command succeeds with `--noproxy`, the Ollama service is healthy and
the failure is caused by proxy routing:

```bash
curl --noproxy 127.0.0.1,localhost -i http://127.0.0.1:11434/api/tags
```

Recommended shell configuration:

```bash
export NO_PROXY=127.0.0.1,localhost,::1,$NO_PROXY
export no_proxy=127.0.0.1,localhost,::1,$no_proxy
```

Place these lines after `HTTP_PROXY`, `HTTPS_PROXY`, `http_proxy`, or
`https_proxy` are set, for example near the end of `~/.bashrc`.

Validation:

```bash
curl -i http://127.0.0.1:11434/api/tags
```

Expected result:

```text
HTTP/1.1 200 OK
```

Do not treat a local Ollama `503` as a model failure until proxy bypass,
service status, and `/api/tags` have been checked.

## Recommended Initial Role

Initial Hermes use should be limited to:

```text
User / logs / evidence / artifacts
  -> Ollama text preprocessing
  -> Hermes Primary validates and routes
  -> Codex planning or review when needed
  -> CodeBuddy execution when needed
```

Do not use Ollama to replace Codex or CodeBuddy. Use it to reduce formatting,
summarization, and cleanup cost before authority-bearing workflow phases.

## Evidence

Every Ollama-assisted phase should record:

- model name
- observed context length, when available
- estimated input size
- whether the context budget gate passed
- prompt purpose
- output role, such as `draft_summary` or `candidate_checklist`
- whether Hermes Primary accepted, edited, rejected, or escalated the output

If context size is unavailable, record `estimated_input_tokens: unavailable`
and prefer routing to Hermes Primary for anything larger than a short bounded
packet.
