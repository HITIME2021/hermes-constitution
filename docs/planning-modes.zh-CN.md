# Planning Modes 规划模式

Planning Modes 定义 Hermes 在 live execution 前，从哪里获得一个任务的 planning
source of record。

Codex 仍然是默认 planner 和 senior reviewer。Frontend artifact tools 可以在复杂
或模糊任务中辅助规划，但不会替代 Hermes authority，也不会取消 Codex review
责任。

## 核心规则

<!-- snapshot:block id="planning-modes.zh-CN" section="Planning Modes" priority="83" -->
Hermes 每个任务必须选择一个 planning source of record：

```yaml
planning_source_of_record: codex_native | frontend_artifact_assisted
```

`codex_native` 是默认模式。Hermes 请求 Codex CLI 生成 Hermes 原生 planning
packet：

```text
Task
ExecutionRequest
ReviewPlan
Stop Conditions
Evidence Requirements
```

小型或局部任务应使用 `codex_native`，包括 bugfix、测试补充、README/docs 修改、
CLI 输出优化、小型 utility 修改，以及 Hermes 已熟悉代码库中的局部工作。

`frontend_artifact_assisted` 是可选模式，用于复杂、模糊或产品形态较强的工作。
Hermes 可以请求一个或多个 Frontend Tools 生成 advisory artifacts，例如
specification、plan、task list、checklist、design brief 或 clarifying questions。
这些 artifacts 是 input evidence，不是 authority。

使用 `frontend_artifact_assisted` 时，Codex 不应从零重复完整 planning。Codex 负责
review 和 normalize artifacts，映射风险，识别缺失澄清，并帮助 Hermes 把 artifact
set 转换成 Hermes 原生 `Task`、`ExecutionRequest`、`ReviewPlan`、stop conditions
和 evidence requirements。

Hermes 必须单独记录具体 frontend tool：

```yaml
planning_source_of_record: frontend_artifact_assisted
frontend_tool_name: example-tool
frontend_tool_mode: artifact_generation_only
```

最终 live-execution authority 永远是 Hermes `ExecutionRequest`。Frontend artifacts、
Codex planning notes 和 review notes 在 Hermes validate、normalize、map 并通过
approval gate 前，都只是 evidence 和 input。
<!-- /snapshot:block -->

## 模式选择

默认使用 `codex_native`，除非有明确理由使用 frontend artifact assistance。

以下情况使用 `codex_native`：

- 任务小而局部
- 目标代码路径已知
- 任务影响一到两个文件
- acceptance criteria 已清楚
- operator request 可以直接表达成 scoped `ExecutionRequest`
- 完整 specification 只会增加流程成本，而不能降低风险

以下情况可以考虑 `frontend_artifact_assisted`：

- 用户请求很长、不清楚，或产品形态较强
- 任务引入新模块、新 workflow、新 UI 或新 user journey
- 工作影响三个以上模块
- 任务需要明确产品行为、信息架构或 UX 决策
- Codex native planning 返回 `requires_clarification`
- 同一需求需要超过一轮澄清
- operator 明确要求执行前先产出正式 specification

## Codex 在不同模式下的角色

```text
codex_native
  Codex role: planner + risk evaluator + reviewer
  Codex output: Hermes-native planning packet

frontend_artifact_assisted
  Codex role: artifact reviewer + normalizer + risk mapper
  Codex output: artifact review, gap analysis, normalized Hermes packet
```

在 `frontend_artifact_assisted` 中，Codex 应重点判断 artifacts 是否清楚、安全、
scope 合理、可测试，并能否映射到 Hermes execution。除非 artifact set 缺失或不安全，
Codex 不应花 token 重新创建一份并行完整 specification。

## 避免重复规划

Hermes 必须避免 double-planning：

```text
Do not ask a Frontend Tool to create a full plan and then ask Codex to create
another full plan from scratch for the same task.
```

如果 artifact set 不足，Codex 应返回：

```text
requires_clarification
artifact_gap
scope_gap
risk_gap
not_mappable_to_execution_request
```

Hermes 随后请求澄清或 artifact revision，而不是进入 live execution。

## Evidence

Planning evidence 应记录：

```yaml
planning_mode:
  planning_source_of_record: codex_native
  frontend_tool_name: null
  frontend_tool_mode: null
  reason_selected: "small local bugfix"
  codex_role: planner
```

Artifact-assisted planning：

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

Planning mode selection 也应写入 final report 和 audit-grade evidence。

## 非目标

Planning Modes 不授予 Frontend Tools 执行权威。

Planning Modes 不降低 Review Gate 要求。Medium 及以上风险任务仍必须按策略具备
Codex planning 和/或 Codex review。

Planning Modes 不绕过 operator approval、scope control、dependency approval、
memory policy 或 stop conditions。
