# Runtime and Provider Interfaces

Hermes core should be headless.

GUI is useful, but it must not be the engine of the automation system. The core
runtime should be able to run in WSL, on a server, in CI, or behind a future web
or desktop control plane.

## Runtime Policy

```yaml
runtime_policy:
  core_mode: headless
  primary_interface: cli
  secondary_interface: api
  gui_role: optional_control_plane
```

## Why Headless

Headless core makes Hermes easier to:

- automate
- test
- retry
- audit
- run in WSL
- run in CI
- connect to provider adapters
- recover after failure
- expose through future GUI, web, chat, or IDE surfaces

If Hermes depends on GUI automation for its core workflow, it becomes harder to
debug, parallelize, and deploy.

## Interface Roles

```text
Hermes Core
  headless runtime, workflow, policy, state, memory, routing

CLI
  primary automation interface

API
  service interface for future integrations

GUI / Desktop
  optional human control plane
```

## Human Control Plane

GUI or desktop software is still valuable, but only as a control plane.

It should be used for:

- viewing task state
- reviewing diffs
- approving high or critical risk operations
- debugging failed tasks
- inspecting memory candidates
- collaborating with Codex interactively

It should not be required for:

- task planning
- provider routing
- CodeBuddy execution
- Codex review
- state transitions
- memory writeback

## Provider Interface Policy

Providers should be integrated through adapters.

```text
Hermes Workflow Engine
  -> Capability Resolver
  -> Provider Adapter
  -> Provider CLI/API
  -> Structured Result
  -> Review Gate
```

Hermes should not bind its core logic directly to a provider UI.

Provider adapters are responsible for transport reliability and error
classification. A CLI/API timeout must be reported as `transport_error`, not as
a semantic task failure.

```text
CLI timeout
  -> Provider Adapter
  -> transport_error
  -> bounded retry or blocked
  -> no task replanning by default
```

## Codex Provider

Codex is the senior brain.

```yaml
codex_provider:
  role: brain_and_senior_engineer
  primary_interface: cli
  secondary_interface: api
  gui_interface: optional_human_collaboration
  default_functions:
    - planning
    - architecture_design
    - algorithm_design
    - risk_evaluation
    - review
    - replanning
    - memory_synthesis
```

Codex desktop can be used by the human operator for deep design, review, and
interactive investigation. Hermes automation should call Codex through CLI/API.

## CodeBuddy Provider

CodeBuddy is the low-cost executor.

```yaml
codebuddy_provider:
  role: low_cost_scoped_executor
  primary_interface: cli
  secondary_interface: api
  gui_interface: optional_debugging
  default_functions:
    - scoped_code_editing
    - code_generation
    - repetitive_refactor
    - simple_bugfix
    - test_writing
```

CodeBuddy should receive scoped execution packets and return structured
execution results. It should not depend on GUI interaction for normal
automation.

## Recommended CLI Shape

Future Hermes CLI can expose commands like:

```bash
hermes plan "Add loading state to SubmitButton"
hermes route task-001
hermes execute task-001 --provider codebuddy
hermes review task-001
hermes memory writeback task-001
```

Provider adapters can call external CLIs:

```text
codex CLI/API
codebuddy CLI/API
```

The provider-specific command syntax should stay inside the adapter.

## Boundary Rule

```text
Hermes controls state.
Provider performs work.
Review Gate decides quality.
GUI helps humans supervise.
```

Provider transport failure is handled by the adapter and ExecutionAttempt
policy. It must not be allowed to masquerade as architecture failure.
