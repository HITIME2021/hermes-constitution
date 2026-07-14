# Planning Modes

Planning Modes define where Hermes obtains the planning source of record for a
task before live execution.

Codex remains the default planner and senior reviewer. Frontend artifact tools
may assist planning for complex or ambiguous work, but they do not replace
Hermes authority or Codex review responsibility.

## Core Rule

<!-- snapshot:block id="planning-modes" section="Planning Modes" priority="82" -->
Hermes must choose one planning source of record per task:

```yaml
planning_source_of_record: codex_native | frontend_artifact_assisted
```

`codex_native` is the default mode. Hermes asks Codex CLI to produce the native
planning packet:

```text
Task
ExecutionRequest
ReviewPlan
Stop Conditions
Evidence Requirements
```

Use `codex_native` for small or local tasks, including bug fixes, test
additions, README or docs edits, CLI output tweaks, small utility changes, and
work inside a codebase Hermes already understands.

`frontend_artifact_assisted` is an optional mode for complex, ambiguous, or
product-shaped work. Hermes may ask one or more Frontend Tools to produce
advisory artifacts such as specifications, plans, task lists, checklists,
design briefs, or clarifying questions. These artifacts are input evidence, not
authority.

When `frontend_artifact_assisted` is used, Codex must not duplicate full
planning from scratch. Codex reviews and normalizes the artifacts, maps risk,
identifies missing clarification, and helps Hermes convert the artifact set
into Hermes-native `Task`, `ExecutionRequest`, `ReviewPlan`, stop conditions,
and evidence requirements.

Hermes must record the concrete frontend tool separately:

```yaml
planning_source_of_record: frontend_artifact_assisted
frontend_tool_name: example-tool
frontend_tool_mode: artifact_generation_only
```

The final live-execution authority is always the Hermes `ExecutionRequest`.
Frontend artifacts, Codex planning notes, and review notes are evidence and
inputs until Hermes validates, normalizes, maps, and passes the approval gate.
<!-- /snapshot:block -->

## Mode Selection

Default to `codex_native` unless there is a concrete reason to use frontend
artifact assistance.

Use `codex_native` when:

- the task is small or local
- the target code path is already known
- the task affects one or two files
- the acceptance criteria are already clear
- the operator request can be expressed directly as a scoped
  `ExecutionRequest`
- a full specification would add process cost without reducing risk

Consider `frontend_artifact_assisted` when:

- the user request is long, unclear, or product-shaped
- the task introduces a new module, workflow, UI, or user journey
- the work affects three or more modules
- the task needs explicit product behavior, information architecture, or UX
  decisions
- Codex native planning returns `requires_clarification`
- the same request requires more than one clarification round
- the operator explicitly asks for a formal specification before execution

## Codex Role by Mode

```text
codex_native
  Codex role: planner + risk evaluator + reviewer
  Codex output: Hermes-native planning packet

frontend_artifact_assisted
  Codex role: artifact reviewer + normalizer + risk mapper
  Codex output: artifact review, gap analysis, normalized Hermes packet
```

In `frontend_artifact_assisted`, Codex should focus on whether the artifacts are
clear, safe, scoped, testable, and mappable to Hermes execution. It should not
spend tokens recreating a parallel full specification unless the artifact set is
missing or unsafe.

## Duplication Guard

Hermes must avoid double-planning:

```text
Do not ask a Frontend Tool to create a full plan and then ask Codex to create
another full plan from scratch for the same task.
```

If the artifact set is insufficient, Codex should return:

```text
requires_clarification
artifact_gap
scope_gap
risk_gap
not_mappable_to_execution_request
```

Hermes then asks for clarification or artifact revision before live execution.

## Evidence

Planning evidence should record:

```yaml
planning_mode:
  planning_source_of_record: codex_native
  frontend_tool_name: null
  frontend_tool_mode: null
  reason_selected: "small local bugfix"
  codex_role: planner
```

For artifact-assisted planning:

```yaml
planning_mode:
  planning_source_of_record: frontend_artifact_assisted
  frontend_tool_name: example-tool
  frontend_tool_mode: artifact_generation_only
  reason_selected: "new multi-step user workflow with unclear scope"
  codex_role: artifact_reviewer_normalizer
  artifacts_reviewed:
    - spec.md
    - plan.md
    - tasks.md
    - checklist.md
```

Planning mode selection should also be referenced in the final report and
audit-grade evidence.

## Non-Goals

Planning Modes do not grant execution authority to Frontend Tools.

Planning Modes do not reduce Review Gate requirements. Medium and higher risk
tasks still require Codex planning and/or Codex review according to policy.

Planning Modes do not bypass operator approval, scope control, dependency
approval, memory policy, or stop conditions.
