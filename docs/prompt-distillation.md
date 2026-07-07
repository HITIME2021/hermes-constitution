# Prompt Distillation Policy

Prompt distillation defines how Hermes turns long operator prompts, chat
history, constitution constraints, and project context into smaller structured
provider packets.

Hermes should optimize for lower context cost and clearer execution contracts
without compressing away facts needed for review, safety, or audit.

## Core Rule

<!-- snapshot:block id="prompt-distillation-policy" section="Prompt Distillation" priority="75" -->
Long prompts are intent evidence, not provider execution contracts. Hermes must
distill long instructions into structured task packets before dispatching work
to providers.

Distillation compresses expression, not facts. Code, commands, paths, schema
fields, error messages, quoted evidence, provider names, and file names must be
preserved exactly when they are policy-relevant or execution-relevant.

Provider packets should include only what the provider needs:

```text
Task
ExecutionRequest
allowed_scope
forbidden_scope
provider_chain
ReviewPlan
Human Intervention Stop Conditions
Evidence Requirements
acceptance_criteria
```

CodeBuddy should receive a scoped execution packet, not the full operator
conversation or full constitution. Codex review should receive the diff,
ReviewPlan, relevant policy excerpts, evidence paths, and enough context to
review safely. It should not receive unrelated history.

Distillation must be honest. Hermes must not claim token savings unless it has
measured or estimated both original and distilled context. If compression makes
operator review, audit, or debugging harder, Hermes should keep the clearer
structure even if it is longer.

Distillation records should include:

```text
distillation_level: normal | compact | terse | audit
source_context_summary
omitted_context_reason
preserved_evidence
provider_packet_summary
estimated_original_context
estimated_distilled_context
```

Distillation must not remove stop conditions, approval requirements, dependency
profile rules, auth boundaries, credential restrictions, review mode, allowed
scope, forbidden scope, or evidence requirements.
<!-- /snapshot:block -->

## Levels

```text
normal
  Default. Preserve clear prose and full structured fields.

compact
  Remove repetition and prior discussion that does not affect execution.

terse
  Use only structured fields and short summaries for high-frequency operations.

audit
  Prefer explicit evidence and traceability over brevity.
```

`audit` mode may be longer than `compact`. It is appropriate for provider
orchestration runs, stop-condition drills, and review reports.

## Non-Goals

Prompt distillation is not a stylistic persona. Hermes must not adopt novelty
speech patterns or rewrite professional operator-facing content into a gimmick.

Prompt distillation is not permission to omit facts. A shorter prompt is worse
if it hides risk, loses evidence, or makes the task harder to review.

## Provider Context Rules

CodeBuddy packet:

- goal
- exact files allowed
- exact files forbidden
- concrete edit instructions
- test command
- report contract
- stop conditions relevant to execution

Codex planning packet:

- user goal
- project facts
- relevant constitution snapshot blocks
- uncertainty and assumptions
- dependency profile
- risk and review requirements

Codex review packet:

- diff
- ReviewPlan
- test results
- evidence paths
- relevant policy excerpts
- known assumptions and residual risk

Never include:

- secrets or credential material
- full unrelated chat history
- full constitution when snapshot blocks are enough
- provider credential files
- `.env` values

