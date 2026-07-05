# Runtime and Provider Interfaces

Hermes core should be headless.

GUI is useful, but it must not be the engine of the automation system. The core
runtime should be able to run in WSL, on a server, in CI, or behind a future web
or desktop control plane.

## Runtime Policy

```yaml
runtime_policy:
  core_mode: headless
  production_execution_plane: wsl
  primary_interface: cli
  secondary_interface: api
  windows_role: human_control_and_constitution_maintenance
  gui_role: optional_human_control_plane
```

## Execution Plane Policy

Hermes production automation should run inside WSL when the project runtime,
package managers, test commands, and provider CLIs are installed there.

For the Windows 11 + WSL operator setup, Hermes treats the environments as two
separate planes:

```text
WSL
  production execution plane
  Hermes Core
  provider adapters
  Codex CLI
  CodeBuddy CLI
  Python / uv
  Node.js / package managers
  tests, lint, build, and project commands

Windows
  human control plane
  Codex Desktop / GUI
  constitution maintenance
  policy review and approval
  interactive debugging and observation
```

Provider CLIs must be installed in the same execution plane as the repository,
dependencies, and test commands they operate on. If `uv`, Node.js, package
manager state, and the production checkout live in WSL, then Codex CLI and
CodeBuddy CLI should also be installed and invoked from WSL.

Hermes must not split a single execution across Windows and WSL. In particular,
CodeBuddy should not edit from Windows while tests run in WSL, and Codex review
should not assume Windows paths when the production repository is a WSL
checkout.

```text
If a task touches implementation, tests, dependencies, provider execution, or
automation state, execute it in WSL.

If a task touches constitution design, policy review, approval, or human
collaboration, Windows may be used as the control plane.
```

Provider adapters must resolve paths, commands, dependency assumptions, and
runtime evidence against the WSL workspace for production execution.

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

## Provider Auth Policy

Provider authentication is owned by the operator, not Hermes automation.

For the Windows 11 + WSL operator setup, Codex CLI should use the user's WSL
ChatGPT/OAuth session created by `codex login`. Hermes must not inject
`OPENAI_API_KEY` into Codex execution by default when the operator intends to use
ChatGPT/Plus entitlement.

Hermes may verify provider readiness through non-secret checks such as provider
path, version, `codex doctor`, and explicitly approved smoke tests. It must not
read, copy, store, or synthesize tokens, API keys, session cookies, credential
files, or `.env` values.

See [Provider Auth Policy](provider-auth-policy.md).

## Provider Role vs Model Backend

Hermes provider roles are workflow responsibilities. Model backends are the
underlying LLM services that may power a provider role or auxiliary reasoning
step. They are not the same thing.

```text
provider_role:
  codex     -> senior planning, review, architecture, replanning
  codebuddy -> scoped execution

model_backend:
  deepseek-v4-pro -> optional LLM backend for selected reasoning or summaries
```

DeepSeek-V4-Pro may be available through Hermes configuration, but it is not a
formal Hermes provider in v0.1. It must not replace Codex as the required senior
planning, review, provider-routing, or approval authority unless Project Policy
explicitly promotes it to a `ProviderProfile`.

Allowed default uses for DeepSeek-V4-Pro as a model backend:

- Chinese summarization
- operator-facing explanation drafts
- low-risk document cleanup
- memory candidate drafting before review
- report generation
- non-authoritative classification support

Forbidden default uses unless explicitly approved by policy:

- high-risk planning
- final Codex review
- provider routing decisions
- security, auth, database, infrastructure, or deployment decisions
- automatic approval
- direct long-term memory writeback
- direct provider execution

If DeepSeek output influences a decision that affects execution, risk, review,
memory, or approval, Hermes must record it as supporting evidence, not as the
authoritative decision source.

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
