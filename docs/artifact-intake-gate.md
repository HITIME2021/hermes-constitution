# Artifact Intake Gate

Artifact Intake Gate defines how Hermes receives, validates, normalizes, and
maps artifacts produced by Frontend Tools before those artifacts can influence
execution.

Frontend artifacts can improve planning quality, reduce context waste, and make
ambiguous work easier to review. They are still input evidence, not authority.

## Core Rule

<!-- snapshot:block id="artifact-intake-gate" section="Artifact Intake" priority="84" -->
Frontend artifacts are not execution authority. Hermes must intake artifacts
before they can influence live execution.

Artifact Intake Gate:

```text
1. validate source and tool mode
2. classify artifact type
3. check freshness and target project alignment
4. detect hidden effects, shell steps, or mutation assumptions
5. normalize into Hermes-native fields
6. map to Task / ExecutionRequest / ReviewPlan / Stop Conditions
7. require Codex artifact review when risk, ambiguity, or scope exceeds threshold
8. pass operator approval before live execution
```

Artifact intake must preserve the distinction between evidence and authority:

```text
artifact evidence -> Hermes-native draft -> review/approval -> ExecutionRequest
```

Hermes must reject or stop intake when an artifact:

- includes unapproved shell execution or mutation steps
- assumes provider routing authority
- treats itself as approval for live execution
- expands scope silently
- references files, symbols, commands, or behavior not present in the selected
  target worktree and commit
- omits stop conditions for non-trivial work
- conflicts with the loaded constitution snapshot
- requests dependency, credential, auth, provider configuration, deployment, or
  memory writeback authority without explicit approval
- cannot be mapped to a bounded `ExecutionRequest`

If an artifact is useful but incomplete, Hermes must mark it as draft evidence
and request clarification, artifact revision, or Codex artifact review before
live execution.
<!-- /snapshot:block -->

## Artifact Types

Common artifact mappings:

```text
spec.md       -> Task, user intent, acceptance criteria
plan.md       -> ExecutionRequest draft, scope draft, risk notes
tasks.md      -> provider task packet drafts
checklist.md  -> ReviewPlan, verification checklist, stop conditions
risk.md       -> RiskEvaluation input
design.md     -> product behavior and UX constraints
clarify.md    -> open questions, unresolved assumptions
```

These mappings are drafts. Hermes must normalize field names, scope, risk,
provider routing, review mode, and evidence requirements before approval.

## Intake Packet

Hermes should record an intake packet for artifact-assisted planning:

```yaml
artifact_intake:
  source_tool: example-tool
  tool_classification: frontend_tool
  tool_mode: artifact_generation_only
  planning_source_of_record: frontend_artifact_assisted
  target_project: ~/projects/example
  target_worktree: ~/projects/example-hermes-run-001
  target_commit: abc1234
  artifacts:
    - path: spec.md
      type: specification
      status: accepted_as_evidence
    - path: plan.md
      type: plan
      status: requires_codex_review
  rejected_items:
    - item: workflow shell step
      reason: backend_tool_effect_requires_execution_request
  mapped_outputs:
    task: draft
    execution_request: draft
    review_plan: draft
  authority_status: not_approved
```

## Codex Artifact Review

Codex artifact review is required when:

- the work is medium risk or higher
- artifacts define architecture, provider routing, database changes, auth,
  deployment, dependencies, or cross-module behavior
- tasks are split into multiple execution packets
- the artifact includes workflow steps, shell commands, generated code, or
  mutation assumptions
- acceptance criteria are ambiguous
- the artifact conflicts with target worktree evidence or constitution policy

Codex artifact review should answer:

```text
Is the artifact clear?
Is scope bounded?
Is the target premise valid?
Are stop conditions present?
Are hidden effects present?
Can this map to a Hermes ExecutionRequest?
What must be clarified before live execution?
```

Codex artifact review is not live execution approval. Hermes still needs an
operator approval gate before Backend Tools perform effects.

## Premise Alignment

Artifact Intake Gate must use Task Premise Validation for claims about target
files, symbols, commands, tests, configuration, behavior, and bugs.

Artifacts created from stale context, another branch, a dirty working directory,
or a different project may remain useful as background evidence, but they must
not become execution input until the premise is validated against the selected
target worktree and commit.

## Evidence

Audit-grade evidence should include:

```text
artifact_paths
artifact_hashes
source_tool
tool_classification
tool_mode
planning_source_of_record
target_project
target_worktree
target_commit
premise_validation_result
intake_decision
rejected_items
mapping_summary
codex_artifact_review_result
operator_approval_status
```

If artifact content is withheld, summarized, or redacted, Hermes must record
why and preserve enough reference information for audit.

## Non-Goals

Artifact Intake Gate does not execute tools.

Artifact Intake Gate does not approve live execution.

Artifact Intake Gate does not replace Review Gate, Provider Adapter policy,
Human Intervention policy, dependency policy, memory policy, or operator
approval.
