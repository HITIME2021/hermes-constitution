# ADR-0008: Tools Layer and Planning Source of Record

Date: 2026-07-14

Status: Accepted for Hermes Constitution v0.3 draft

## Context

Hermes started with Codex CLI as the senior planning and review brain, and
CodeBuddy CLI as the scoped execution worker. That model remains valid.

As Hermes begins to consider additional tools, some tools can generate
specifications, plans, task lists, checklists, design briefs, or clarifying
questions before execution. Other tools execute commands, edit files, run tests,
call providers, deploy, or mutate state. Treating all tools the same creates
policy ambiguity.

There is also a planning overlap risk. If a frontend artifact tool generates a
full plan and Codex also generates a full plan from scratch for the same task,
Hermes wastes tokens and creates unclear authority boundaries. If Hermes accepts
frontend artifacts directly as execution authority, it bypasses the existing
approval, review, scope, and evidence model.

Hermes therefore needs:

- a tool classification model
- a single planning source of record per task
- an artifact intake gate for frontend outputs
- token telemetry paired with quality telemetry

## Decision

Hermes introduces a Tools Layer classification:

```text
Frontend Tools produce artifacts.
Frontend Tool output is input evidence, not authority.

Backend Tools produce effects.
Backend Tool execution is authority-bearing effect and requires scope control.

Hermes governs both.
```

Tool classification is based on the current usage mode, not the tool brand.

Hermes introduces a planning source of record field:

```yaml
planning_source_of_record: codex_native | frontend_artifact_assisted
```

`codex_native` is the default planning mode. In this mode, Codex CLI remains the
planner, risk evaluator, and reviewer.

`frontend_artifact_assisted` is optional. It is used for complex, ambiguous,
product-shaped, multi-module, or clarification-heavy work where frontend
artifacts can reduce ambiguity or context cost. In this mode, frontend artifacts
are advisory evidence. Codex reviews and normalizes them, maps risk, identifies
gaps, and helps Hermes convert the artifact set into native `Task`,
`ExecutionRequest`, `ReviewPlan`, stop conditions, and evidence requirements.
Codex should not duplicate full planning from scratch unless the artifact set is
missing, unsafe, or not mappable.

Hermes introduces Artifact Intake Gate:

```text
frontend artifact -> validate -> normalize -> map -> review/approval -> ExecutionRequest
```

The final live-execution authority remains the Hermes `ExecutionRequest`.

Hermes introduces Token Telemetry Policy, but token data must be paired with
quality telemetry. Lower token usage is not a success if review quality,
testing, evidence, scope control, or task outcome degrades.

## Consequences

Positive:

- Hermes can use artifact-generating tools without weakening execution control.
- Codex remains the default brain while still allowing frontend artifact
  assistance for complex planning.
- The system avoids double-planning and token waste.
- Frontend artifacts can improve clarity while remaining non-authoritative.
- Backend execution remains bounded by `ExecutionRequest`, scope, approval, and
  evidence.
- Token optimization can be measured without ignoring quality.

Tradeoffs:

- Artifact-assisted planning adds an intake step before execution.
- Hermes must record planning mode and tool classification for runs that use
  additional tools.
- Codex may still need to review artifacts, so frontend tools do not eliminate
  Codex cost entirely.
- Some tools must be split by usage mode, which adds policy complexity.

## Rejected Alternatives

### Let frontend tools directly drive execution

Rejected. Frontend artifacts are input evidence, not authority. Direct execution
would bypass Hermes approval, scope control, review gates, stop conditions, and
evidence rules.

### Replace Codex planning globally with artifact tools

Rejected. Codex remains the default planner and senior reviewer. Artifact tools
are optional assistants for complex or ambiguous planning, not the primary brain
for all tasks.

### Always run both frontend planning and Codex full planning

Rejected. Double-planning wastes tokens and creates competing sources of truth.
Hermes must choose one planning source of record per task.

### Optimize only for token reduction

Rejected. Token telemetry must be paired with quality telemetry. Lower token
usage is not valuable if it creates weaker plans, missed risks, poor review,
incomplete tests, or less auditable evidence.

### Classify tools by brand

Rejected. Tool behavior depends on usage mode. The same tool may be frontend in
artifact-only mode and backend when it executes shell steps, writes files,
installs dependencies, or mutates state.

## Related Documents

- `docs/tools-layer.md`
- `docs/planning-modes.md`
- `docs/artifact-intake-gate.md`
- `docs/token-telemetry-policy.md`
- `docs/provider-adapter-design.md`
- `docs/review-gate.md`
- `docs/human-intervention-policy.md`
