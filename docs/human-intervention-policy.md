# Human Intervention Policy

Hermes is automated, but it must not loop indefinitely. Human intervention is a
control boundary that protects time, tokens, project safety, and strategic
direction.

This policy defines when Hermes must stop automatic retries or replanning and
ask the operator for a decision.

## Core Rule

```text
Automation may retry bounded operational failures.
Automation may revise bounded semantic failures.
Automation must stop when repeated failure suggests missing judgment, missing
context, wrong direction, unsafe scope, or wasteful token burn.
```

When a stop condition is reached, Hermes moves the task to `blocked` and emits a
Human Intervention Request.

## Human Intervention Request

```yaml
human_intervention_request:
  task_id: task-001
  reason: repeated_revision_failure
  current_state: executing
  recommended_next_state: blocked
  summary: "Two CodeBuddy revisions failed the same acceptance criterion."
  attempts:
    execution_attempts: 2
    revision_cycles: 2
    replans: 1
  evidence:
    - "Review found the same validation behavior missing twice."
    - "Full test suite still passes, but acceptance criterion is unmet."
  options:
    - approve_one_more_attempt
    - change_scope
    - ask_codex_to_replan
    - stop_task
  default_recommendation: ask_codex_to_replan
```

## Stop Conditions

Hermes must stop and ask for human intervention when any condition below is
met.

| Condition | Threshold | Required action |
|-----------|-----------|-----------------|
| Same semantic failure repeats | 2 failed revision cycles | Block and ask human |
| Same test failure repeats | 2 attempts with same failing test | Block and ask human |
| Same review finding repeats | 2 reviews with same required action | Block and ask human |
| Replanning repeats without progress | 1 replan already attempted | Ask before another replan |
| Provider transport failures exhaust retry budget | retry budget exhausted | Block, do not replan by default |
| Provider malformed output repeats | 2 malformed responses | Block as provider/adapter issue |
| Scope expansion needed | any forbidden/protected scope request | Ask human or required approver |
| Dependency manifest or lockfile change | any dependency profile match | Ask human |
| Secrets, credentials, auth, db, infra, deployment | any write/change | Ask human; critical may require L4 |
| Acceptance criteria conflict | any conflict | Ask human |
| User intent ambiguous after clarification | 1 failed clarification cycle | Ask human |
| Token/time budget concern | projected extra work exceeds task value | Ask human |
| Generated plan contradicts constitution | any contradiction | Stop and ask human |

<!-- snapshot:block id="dependency-human-intervention" section="Human Intervention" priority="50" -->
Dependency manifest or lockfile changes require human intervention whenever they
match the target project's dependency profile. This must be treated as a
dependency change even when the exact manifest or lockfile name differs by
ecosystem.
<!-- /snapshot:block -->

## Default Budgets

These are v0.1 defaults unless Project Policy defines stricter limits.

```yaml
human_intervention_defaults:
  transport_retry_budget: 2
  semantic_revision_budget: 2
  replanning_budget: 1
  malformed_output_budget: 1
  clarification_budget: 1
  provider_switch_budget: 1
```

Budgets are per task, not per provider message. Hermes must count attempts
across CodeBuddy, Codex, and any future provider when the failure is the same
semantic issue.

## Retry vs Human Intervention

Retries are allowed for bounded operational failures:

- timeout
- network error
- rate limit with backoff
- temporary provider unavailable
- safe malformed response retry

Human intervention is required for judgment failures:

- repeated failed implementation
- repeated review rejection
- unclear or conflicting requirement
- scope needs to expand
- dependency manifest or lockfile changes
- risk becomes higher than originally routed
- provider output is plausible but directionally wrong
- token burn is no longer justified by the expected value

## Token Waste Guard

Hermes must not continue an automatic loop merely because more attempts are
technically possible.

Before any attempt after the first failed semantic revision, Hermes must ask:

- Is the failure the same as before?
- Is new information available?
- Did the plan change meaningfully?
- Is the remaining work still worth another provider call?
- Would a human decision be cheaper than another automated attempt?

If the answer is unclear, stop and ask the operator.

## State Machine Integration

```text
executing -> blocked:
  human intervention stop condition reached

reviewing -> blocked:
  repeated review finding or required approval

replanning -> blocked:
  replan budget exhausted or strategic direction unclear

blocked -> planned:
  human changes scope, acceptance criteria, or plan

blocked -> executing:
  human approves one more bounded attempt

blocked -> failed:
  human stops the task or declares goal unreachable
```

## Hard Rules

- Stop conditions override provider preference.
- Stop conditions override skill confidence.
- Stop conditions override automation momentum.
- CodeBuddy must not continue after repeated failed review without Codex or human decision.
- Codex must not repeatedly replan beyond the replan budget without human approval.
- Human approval must be explicit and recorded as evidence.
- Dry-run may recommend intervention but must not create real approval state.
