# Artifact Intake Gate

Artifact Intake Gate 定义 Hermes 如何接收、验证、规范化并映射 Frontend Tools
产出的 artifacts，之后这些 artifacts 才能影响 execution。

Frontend artifacts 可以提升规划质量、减少上下文浪费，并让模糊工作更容易审查。
但它们仍然是 input evidence，不是 authority。

## 核心规则

<!-- snapshot:block id="artifact-intake-gate.zh-CN" section="Artifact Intake" priority="85" -->
Frontend artifacts 不是 execution authority。Hermes 必须先 intake artifacts，
它们才能影响 live execution。

Artifact Intake Gate：

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

Artifact intake 必须保留 evidence 与 authority 的区别：

```text
artifact evidence -> Hermes-native draft -> review/approval -> ExecutionRequest
```

当 artifact 存在以下情况时，Hermes 必须拒绝或停止 intake：

- 包含未批准的 shell execution 或 mutation steps
- 假定自己拥有 provider routing authority
- 把自身当作 live execution approval
- 静默扩大 scope
- 引用的文件、符号、命令或行为不存在于选定 target worktree / commit
- 非 trivial 工作缺少 stop conditions
- 与已加载 constitution snapshot 冲突
- 未经明确 approval 就请求 dependency、credential、auth、provider configuration、
  deployment 或 memory writeback authority
- 无法映射成有边界的 `ExecutionRequest`

如果 artifact 有用但不完整，Hermes 必须把它标记为 draft evidence，并在 live
execution 前请求 clarification、artifact revision 或 Codex artifact review。
<!-- /snapshot:block -->

## Artifact 类型

常见 artifact 映射：

```text
spec.md       -> Task, user intent, acceptance criteria
plan.md       -> ExecutionRequest draft, scope draft, risk notes
tasks.md      -> provider task packet drafts
checklist.md  -> ReviewPlan, verification checklist, stop conditions
risk.md       -> RiskEvaluation input
design.md     -> product behavior and UX constraints
clarify.md    -> open questions, unresolved assumptions
```

这些映射都是 draft。Hermes 必须在 approval 前规范化 field names、scope、risk、
provider routing、review mode 和 evidence requirements。

## Intake Packet

Hermes 应为 artifact-assisted planning 记录 intake packet：

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

以下情况必须进行 Codex artifact review：

- 工作是 medium risk 或更高
- artifacts 定义 architecture、provider routing、database changes、auth、
  deployment、dependencies 或 cross-module behavior
- tasks 被拆成多个 execution packets
- artifact 包含 workflow steps、shell commands、generated code 或 mutation assumptions
- acceptance criteria 模糊
- artifact 与 target worktree evidence 或 constitution policy 冲突

Codex artifact review 应回答：

```text
Is the artifact clear?
Is scope bounded?
Is the target premise valid?
Are stop conditions present?
Are hidden effects present?
Can this map to a Hermes ExecutionRequest?
What must be clarified before live execution?
```

Codex artifact review 不是 live execution approval。Backend Tools 产生 effects 前，
Hermes 仍然需要 operator approval gate。

## Premise Alignment

Artifact Intake Gate 必须对 target files、symbols、commands、tests、
configuration、behavior 和 bugs 的声明使用 Task Premise Validation。

来自过期上下文、其他分支、dirty working directory 或不同项目的 artifacts 可以作为
background evidence 保留，但在针对选定 target worktree / commit 验证 premise 前，
不得成为 execution input。

## Evidence

Audit-grade evidence 应包含：

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

如果 artifact 内容被 withheld、summarized 或 redacted，Hermes 必须记录原因，并保留
足够 reference information 供 audit。

## 非目标

Artifact Intake Gate 不执行工具。

Artifact Intake Gate 不批准 live execution。

Artifact Intake Gate 不替代 Review Gate、Provider Adapter policy、Human
Intervention policy、dependency policy、memory policy 或 operator approval。
