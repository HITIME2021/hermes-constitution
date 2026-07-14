# Provider Adapter Design

Provider adapters are the boundary between Hermes workflow protocol and
provider-native CLI or API calls.

The adapter layer exists so Hermes Core can stay headless, auditable, and
provider-neutral. Hermes owns task state and policy decisions. Providers perform
assigned work and return structured results.

## Goals

- translate Hermes `ExecutionRequest` objects into provider-native calls
- collect provider output and normalize it into `ExecutionResult`
- classify transport, provider, and execution failures before Workflow Engine
  sees them
- enforce provider scope boundaries
- preserve enough evidence for review, retry, and memory synthesis
- avoid binding Hermes Core to any provider GUI

## Non-Goals

- make provider-specific architectural decisions
- mutate Hermes task state directly
- bypass Project Policy, Risk Policy, or Review Gate
- hide provider failures behind unstructured text
- treat CLI/API transport failures as semantic task failures

## Interface Contract

Every provider adapter should expose the same logical operations:

```text
submit_execution(request) -> adapter_submission
get_status(external_execution_id) -> adapter_status
collect_result(external_execution_id) -> execution_result
cancel(external_execution_id) -> cancel_result
```

Adapters may implement these operations through CLI, API, local process control,
or a test double. The provider-specific command shape must stay inside the
adapter.

## Provider Role and Model Backend Separation

Provider adapters bind Hermes workflow provider roles to executable transport.
A model backend is not automatically a provider adapter.

DeepSeek-V4-Pro may be configured as an LLM backend for low-risk Chinese
summarization, explanation drafting, report generation, or memory candidate
drafting. That configuration does not create a `deepseek` Hermes provider and
does not make DeepSeek eligible for `assigned_provider` in `ExecutionRequest`.

In v0.1, `assigned_provider` remains limited to formal provider roles such as
`codex` and `codebuddy`. If Hermes needs DeepSeek to become a provider, the
constitution must first define a ProviderProfile, risk policy, adapter contract,
failure classification, review authority, and memory permissions for it.

Adapters must reject execution requests that route protected decisions to a
model backend as if it were a formal provider.

## Adapter Submission

```yaml
adapter_submission:
  execution_id: exec-001
  provider: codebuddy
  external_execution_id: codebuddy-run-921
  status: accepted
  started_at: "2026-07-03T00:00:00Z"
  attempt_id: attempt-001
```

If the provider cannot be reached, the adapter returns a classified
`ProviderError` instead of an accepted submission.

## Status Mapping

Provider-native statuses must be mapped into Hermes attempt statuses:

```text
queued       -> pending
running      -> running
completed    -> succeeded
timeout      -> transport_failed
cli_error    -> transport_failed or provider_failed
tool_error   -> provider_failed
test_failed  -> execution_failed
cancelled    -> cancelled
```

The adapter must include raw provider evidence when status mapping is not
obvious.

## Error Classification

Adapters classify failures before returning them to Workflow Engine.

```yaml
provider_error:
  category: transport_error
  type: timeout
  provider: codex
  retryable: true
  counts_as_task_failure: false
  counts_as_provider_reliability_signal: true
  should_trigger_replanning: false
  evidence:
    - "Codex CLI timed out after 120 seconds."
  recommended_action: retry_with_backoff
```

Categories:

```text
transport_error
  Provider channel failed. Examples: timeout, network_error, rate_limit,
  auth_error, provider_unavailable, cli_crash, malformed_response.

provider_error
  Provider process or adapter failed for provider-local reasons. Examples:
  tool_unavailable, sandbox_setup_failed, adapter_protocol_mismatch.

execution_error
  Assigned work semantically failed. Examples: test_failure,
  implementation_failed, review_failed, scope_conflict, missing_context,
  invalid_assumption.
```

Transport errors do not trigger task replanning by default.

## Scope Enforcement

Before submitting a request, the adapter validates:

- assigned provider matches the request
- requested files are inside `allowed_scope`
- requested commands are inside `allowed_scope`
- forbidden files and actions are not present
- provider risk limits are not exceeded
- approval-required operations include approval evidence
- secrets are not included in provider context

The adapter may reject the request with `policy_violation` if it cannot safely
submit it.

<!-- snapshot:block id="provider-self-edit-cost-boundary" section="Provider Adapter" priority="66" -->
## Hermes Self-Edit Disabled by Default

Hermes is the orchestrator, not the default coder. Local Hermes agent
customizations may be necessary, but self-editing through Hermes' own model
backend is disabled by default.

Default routing for code changes:

```text
Hermes planning -> CodeBuddy scoped execution -> verification -> Codex review
```

Hermes must not directly patch, rewrite, create, or delete code files through
its own model/tool loop for implementation work. This applies even when the
target is a local Hermes agent customization.

Hermes self-edit may occur only under an explicit emergency override from the
operator. The override must be task-specific and must include all of:

- the target is a local operator customization, not an upstream contribution
- the scope is small, explicit, and limited to local files
- no auth, credential, provider configuration, memory, dependency, database,
  deployment, or security boundary is touched
- no git commit, push, or pull request will be created
- tests or smoke checks are defined before the edit
- the operator explicitly states that Hermes self-edit is allowed for this task
- evidence records that this was an emergency local self-edit override

Without that explicit override, Hermes must route implementation work to
provider orchestration.

Hermes self-edit is forbidden, even with a broad local customization request,
when:

- the change affects input dispatch, shell execution, auth, memory, provider
  routing, command approval, filesystem mutation, or other safety boundary
- the fix requires repeated attempts
- the implementation begins to consume substantial model context or token cost
- CodeBuddy is available and the task is a scoped code edit
- Codex adversarial review is required by risk level

DeepSeek-V4-Pro or any other model backend used by Hermes does not become the
default coding provider. Cost-sensitive scoped implementation should prefer the
formal executor provider, normally CodeBuddy, unless Project Policy or operator
approval says otherwise.

Local Hermes agent customizations are personal operator changes by default.
They must not be committed, pushed, or proposed upstream unless the operator
explicitly changes that policy for the specific task.
<!-- /snapshot:block -->

## Auth Boundary Enforcement

Provider adapters may use existing operator-authorized CLI sessions, but they
must not manage credential material.

For Codex in this operator environment, the default auth path is WSL
ChatGPT/OAuth login through `codex login`. Adapters must not default to
`OPENAI_API_KEY` for Codex execution when the operator intends to use
ChatGPT/Plus entitlement.

Adapters must reject or block requests that require reading token files, copying
credentials between surfaces, injecting API keys, or passing auth material to
another provider. Auth failures are classified as transport/provider readiness
issues and move the task to `blocked`, not semantic task failure.

<!-- snapshot:block id="provider-interactive-confirmation-boundary" section="Provider Adapter" priority="65" -->
## Interactive Confirmation Boundary

Provider CLIs may ask for interactive confirmation during execution, such as:

```text
trust directory
allow command
apply patch
continue despite risk
authenticate or log in
install dependency
use network
write outside sandbox
approve elevated permission
```

Hermes must not auto-confirm prompts that approve new trust, auth, dependency,
network, destructive, elevated, persistent setting, or scope-expanding actions.

Hermes may use non-interactive scoped execution flags such as `-y` or `--yes`
only when all conditions are true:

- the operator already approved live execution for the exact ExecutionRequest
- `allowed_scope`, `forbidden_scope`, provider, risk, and verification commands
  are explicit
- the provider action is limited to scoped edit/test execution inside
  `allowed_scope`
- the action does not approve auth/login, dependency install, network access,
  destructive operations, sudo/elevated permission, git commit/push, provider
  settings changes, credential access, or scope expansion
- the final evidence records `preapproved_noninteractive_execution: true`, the
  bypass flag used, and the exact approval scope

If a provider CLI requires confirmation, stdin/TTY interaction, or returns a
confirmation-related failure outside the pre-approved scoped execution rules,
the adapter must stop and return a classified result:

```yaml
provider_error:
  category: provider_error
  type: needs_user_confirmation
  provider: codex | codebuddy
  retryable: false
  counts_as_task_failure: false
  should_trigger_replanning: false
  recommended_action: ask_operator
```

The operator-facing request must include:

```text
provider
prompt_type
requested_action
risk
exact command or operation, when safe
recommended_response
whether approval would expand scope, permissions, dependencies, or auth state
```

Unknown confirmation prompts are treated as unsafe until the operator decides.
Hermes must not keep retrying a confirmation prompt as a transport failure.
Pre-approved non-interactive scoped execution is an efficiency rule, not a
permission expansion.
<!-- /snapshot:block -->

## Execution Plane Enforcement

Adapters must execute in the same production plane as the project checkout,
runtime dependencies, and test commands.

For the Windows 11 + WSL operator setup:

```text
WSL = production execution plane
Windows = human control and constitution maintenance plane
```

If the repository, Python/uv environment, Node.js environment, package manager
state, and validation commands live in WSL, then Codex CLI and CodeBuddy CLI
must also be installed and invoked from WSL for automated execution.

Adapters must reject or block execution requests that would split one task
across Windows and WSL in a way that changes path semantics, dependency state,
line endings, permissions, shell behavior, or test evidence.

## Codex Adapter

Codex is the senior reasoning provider.

Default use cases:

- planning
- risk evaluation
- architecture and algorithm design
- code review
- failure analysis
- replanning
- memory synthesis

Codex adapter requirements:

- provide global reasoning context when authorized
- preserve planning and review rationale in structured output
- classify CLI/API timeout as `transport_error`
- never treat malformed output as task failure without evidence
- support review-only requests that do not modify files

## CodeBuddy Adapter

CodeBuddy is the scoped execution provider.

Default use cases:

- file-level implementation
- repetitive edits
- small bug fixes
- test writing
- low-risk refactors

CodeBuddy adapter requirements:

- submit only bounded execution packets
- include allowed and forbidden scope explicitly
- reject protected-domain writes unless scoped approval is present
- require a structured report contract
- capture changed files and diff summary
- route all medium and higher risk results through Codex review

## Retry Policy

Retries belong to `ExecutionAttempt`, not task replanning.

```text
timeout with retry budget -> new ExecutionAttempt
network_error with retry budget -> new ExecutionAttempt
rate_limit -> delayed retry or allowed fallback
auth_error -> blocked
provider_unavailable after retry budget -> blocked
execution_error -> Review Gate or Codex diagnosis
```

Adapters must not retry irreversible operations unless the execution request is
explicitly idempotent.

## Evidence Requirements

Every adapter result should include:

- provider name
- execution id
- attempt id
- status
- timestamps
- command or API operation summary
- files changed, when applicable
- tests or checks run, when applicable
- raw error summary, when applicable
- classification rationale for failures

Evidence should be sufficient for Review Gate and Memory Center without leaking
secrets.

## Boundary Rule

```text
Hermes controls state.
Provider adapter controls transport.
Provider performs work.
Review Gate decides quality.
Memory Center stores reusable lessons.
```
