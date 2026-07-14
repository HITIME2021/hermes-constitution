# Token Telemetry Policy

Token telemetry records provider and tool context usage for audit, cost
awareness, prompt optimization, and planning-mode evaluation.

Token telemetry helps Hermes understand whether a run was efficient. It must not
be used alone to judge whether a run was good.

## Core Rule

<!-- snapshot:block id="token-telemetry-policy" section="Token Telemetry" priority="86" -->
Token telemetry is audit evidence, not billing truth unless provider-reported.
Hermes must not fabricate token counts.

Every provider or tool phase should record token telemetry when available. If
the provider or tool does not expose token usage, Hermes must record the field
as `unavailable`. If Hermes estimates usage, the source must be marked
`estimated`.

Token telemetry must be paired with quality telemetry. Hermes must not optimize
for lower token use alone. Planning-mode comparison, provider selection, prompt
distillation, and frontend artifact assistance must consider both cost and
quality:

```text
token usage + task quality + review verdict + regression result + evidence completeness
```

Task final reports should include a compact token/quality table when telemetry
exists, and should explicitly say when token data is unavailable.

Telemetry source values:

```text
cli_reported
api_reported
estimated
unavailable
```

Rules:

- use `cli_reported` only when a CLI reports token usage directly
- use `api_reported` only when an API response reports token usage directly
- use `estimated` only when Hermes explicitly estimates usage
- use `unavailable` when usage cannot be obtained
- never present an estimate as provider-reported usage
- never infer exact token counts from latency, cost feeling, or model name
- never record raw secrets, credentials, tokens, cookies, or full private prompts
- preserve enough prompt summary and evidence references to support audit
- final reports must include quality outcome, not only token usage
<!-- /snapshot:block -->

## Telemetry File

Recommended path:

```text
~/.hermes/kanban/logs/<board>/<task_id>/<run_id>/token-telemetry.json
```

The telemetry file should be referenced from `evidence.txt` and the final
report.

## Schema

```yaml
token_telemetry:
  task_id: t_example
  run_id: run-001
  constitution_version: abc1234
  planning_source_of_record: codex_native
  total:
    prompt_tokens: null
    completion_tokens: null
    total_tokens: null
    estimated_cost: null
    source: unavailable
  provider_chain:
    - phase: codex_planning
      provider: codex
      tool_classification: backend_tool
      model: gpt-5.5
      prompt_tokens: null
      completion_tokens: null
      total_tokens: null
      estimated_cost: null
      source: unavailable
      evidence_ref: evidence.txt#codex_planning
    - phase: codebuddy_execution
      provider: codebuddy
      tool_classification: backend_tool
      model: deepseek-v4-pro
      prompt_tokens: null
      completion_tokens: null
      total_tokens: null
      estimated_cost: null
      source: unavailable
      evidence_ref: evidence.txt#codebuddy_execution
  prompt_distillation:
    original_context_estimate: null
    distilled_context_estimate: null
    compression_ratio: null
    omitted_context_reason: null
    preserved_evidence:
      - allowed_scope
      - forbidden_scope
      - stop_conditions
  quality:
    final_status: pass
    review_verdict: approved_with_notes
    tests_result: "247/247 passed"
    regression_result: none_detected
    evidence_completeness: audit_grade
    stop_conditions_triggered: []
```

Use `null` for numeric values that are unavailable. Do not use `0` unless the
provider actually reported zero.

## Final Report Table

When available, final reports should include a compact table:

```text
phase | provider | model | tokens | source | quality signal
```

Example:

```text
codex_planning      | codex    | gpt-5.5        | unavailable | unavailable | plan accepted
codebuddy_execution | codebuddy | deepseek-v4-pro | unavailable | unavailable | scoped diff valid
codex_review        | codex    | gpt-5.5        | unavailable | unavailable | approved_with_notes
```

If data is unavailable, the report should say so plainly instead of omitting the
row.

## Quality Signals

Token data must be interpreted with quality signals:

- final status
- review verdict
- tests run and result
- regression status
- stop conditions triggered
- number of bounded revisions
- evidence completeness
- operator intervention count
- whether planning mode was appropriate

Lower token usage is not a success if the run has weak evidence, poor review,
missing tests, hidden scope expansion, or unresolved defects.

## Planning Mode Evaluation

When comparing `codex_native` and `frontend_artifact_assisted`, Hermes should
compare:

```text
planning tokens
review tokens
execution retries
clarification rounds
review findings
test/regression outcome
evidence completeness
operator intervention count
time to approved ExecutionRequest
```

`frontend_artifact_assisted` should be considered beneficial only when it
reduces overall cost or ambiguity without reducing task quality, review quality,
or auditability.

## Privacy and Redaction

Telemetry must not include:

- raw credentials
- API keys
- auth tokens
- cookies
- `.env` values
- full private prompts when a summary is enough
- provider home credential/config file contents

If prompt content is summarized instead of recorded, the telemetry should state:

```yaml
redaction:
  raw_prompt_recorded: false
  reason: private_context_or_unnecessary_for_audit
  summary_ref: evidence.txt#prompt_summary
```

## Non-Goals

Token telemetry is not a billing system.

Token telemetry is not a reason to skip Codex review, approval gates, stop
conditions, or evidence.

Token telemetry must not push Hermes to prefer cheap but low-quality execution
when task risk requires stronger planning or review.
