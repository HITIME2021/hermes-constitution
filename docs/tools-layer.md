# Tools Layer

The Tools Layer defines how Hermes treats external tools that help with
planning, artifact generation, execution, verification, review, and reporting.

Hermes may use many tools, but tool output is not automatically policy
authority. Hermes remains the orchestrator, policy owner, and final execution
authority.

## Core Rule

<!-- snapshot:block id="tools-layer-classification" section="Tools Layer" priority="80" -->
Hermes classifies tools by the kind of result they produce:

```text
Frontend Tools produce artifacts.
Frontend Tool output is input evidence, not authority.

Backend Tools produce effects.
Backend Tool execution is authority-bearing effect and requires scope control.

Hermes governs both.
```

Frontend Tools are artifact-producing tools. They may help generate
specifications, plans, task lists, checklists, design briefs, risk notes,
clarifying questions, or other operator-facing planning artifacts. Their output
is advisory input. It is not an approved `ExecutionRequest`, not provider
routing authority, not a review verdict, and not memory writeback approval.

Backend Tools are effect-producing tools. They may read project state, execute
commands, edit files, run tests, call provider CLIs or APIs, change runtime
state, deploy, migrate data, or otherwise create observable side effects.
Backend Tools must run only under a Hermes `ExecutionRequest`, with declared
`allowed_scope`, `forbidden_scope`, `execution_mode`, approval state, stop
conditions, and evidence requirements.

Ambiguous Tools default to Backend Tool treatment until Hermes has explicitly
classified the current usage mode as artifact-only. If one tool can both
generate artifacts and execute shell commands or mutations, Hermes must split
the usage modes:

```text
artifact mode  -> Frontend Tool rules
execution mode -> Backend Tool rules
```

Frontend Tool output must pass Hermes intake before execution:

```text
validate -> normalize -> map -> approval gate
```

Hermes must not allow a Frontend Tool to approve work, route providers, bypass
review, bypass stop conditions, write long-term memory, mutate target project
files, or execute shell effects by implication.

Tool classification is based on the current usage mode, not the tool brand. A
tool used only to generate planning artifacts is treated as a Frontend Tool. The
same tool used to run commands, edit files, call providers, or mutate state is
treated as a Backend Tool for that invocation.
<!-- /snapshot:block -->

## Frontend Tools

Frontend Tools help Hermes and the operator understand what should be built or
reviewed.

Examples:

- specification generators
- design exporters
- document analyzers
- prompt distillers
- checklist generators
- clarification assistants
- project scaffolding tools when used in dry-run or artifact-only mode

Expected outputs:

```text
spec.md
plan.md
tasks.md
checklist.md
design_brief.md
risk_analysis.md
clarifying_questions.md
```

Frontend artifacts may become evidence, context, or draft inputs to Hermes
objects, but they do not become execution authority until Hermes validates and
normalizes them.

## Backend Tools

Backend Tools perform work with observable effects.

Examples:

- Codex CLI when planning, reviewing, or repairing under a task contract
- CodeBuddy CLI when editing code
- shell commands
- `pytest`
- `git`
- `uv`
- `npm`
- database migration tools
- crawler runners
- deployment tools

Backend Tool use must be traceable through the run evidence. When a Backend Tool
needs broader scope, new dependencies, credentials, provider configuration
changes, destructive operations, network-sensitive operations, or interactive
confirmation, Hermes must follow the existing approval and stop-condition rules.

## Tool Examples

Tool examples are illustrative, not special cases. Hermes must classify each
tool invocation by usage mode:

```text
artifact-only specification generation -> Frontend Tool
workflow shell execution               -> Backend Tool
dry-run design export                  -> Frontend Tool
project file generation                -> Backend Tool
read-only inspection command           -> Backend Tool, possibly Simple Shell Direct Mode
```

For example, a specification toolkit can be a Frontend Tool when it only emits
`spec.md`, `plan.md`, `tasks.md`, `checklist.md`, or clarifying questions. If
the same toolkit executes workflow shell steps, creates project files, installs
dependencies, or mutates runtime state, that invocation is Backend Tool
behavior.

## Tool Intake Rules

For every tool invocation, Hermes should classify:

```yaml
tool_classification:
  tool_name: example-tool
  mode: artifact_generation_only
  class: frontend_tool
  expected_outputs:
    - spec.md
    - plan.md
    - tasks.md
  prohibited_effects:
    - live_execution
    - provider_routing
    - approval_decision
    - shell_mutation
    - memory_writeback
```

If the mode changes, the classification must be re-evaluated. A previous
artifact-only classification does not authorize later execution behavior.

## Non-Goals

The Tools Layer does not replace Provider Adapter policy. A tool that executes
provider work still needs provider routing, adapter enforcement, review, and
evidence.

The Tools Layer does not make Frontend Tool artifacts authoritative. Hermes must
still perform artifact intake and operator approval before live execution.
