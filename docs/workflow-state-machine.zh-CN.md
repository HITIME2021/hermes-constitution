# Workflow State Machine 中文说明

Hermes v0.1 采用工程型状态机。它比极简状态机更适合真实开发任务，但又不会像生产部署系统那样过重。

## 主状态

```text
created
clarifying
planned
routed
executing
reviewing
testing
completed
```

## 异常状态

```text
blocked
replanning
failed
```

## 状态解释

### created

Hermes 收到用户目标，并创建 Task ID。

### clarifying

需求不够清楚，需要向用户确认。

### planned

Codex 完成任务拆解、风险初判和验收标准。

### routed

Capability Resolver 已选择角色、技能、Provider 和 Review Level。

### executing

Provider 正在执行任务。明确、低风险、局部化的代码任务通常交给 CodeBuddy。

### reviewing

Review Gate 正在检查执行结果。

### testing

Hermes 运行或验证必要检查，例如 lint、typecheck、unit test、build。

### completed

任务完成，可以交付，并进入 Memory Writeback 判断。

### blocked

当前无法继续，可能缺少信息、权限、环境、用户决策或外部依赖。

### replanning

原计划失败或不再适用，Codex 分析原因并重新规划。

### failed

超过重试次数、目标不可达，或用户终止。

## 状态控制原则

```text
只有 Workflow Engine / Task Planner 可以改变主状态。
Provider 只能提交事件、结果、失败原因或权限请求。
```

Provider 不能自己把任务从 `executing` 改成 `completed`。它只能返回 `execution.completed` 这样的事件，由 Hermes 决定下一状态。

## 常见流转

```text
created -> clarifying -> planned -> routed -> executing -> reviewing -> testing -> completed
```

失败时：

```text
executing -> replanning -> planned
reviewing -> executing
reviewing -> blocked
testing -> replanning
blocked -> planned / routed / executing
replanning -> failed
```

## 为什么需要工程型状态机

Hermes 的目标不是一次性回答问题，而是管理真实任务。真实任务会遇到：

```text
需求不清
Provider 失败
测试失败
权限不足
风险升级
用户审批
上下文缺失
```

工程型状态机让这些情况可以被审计、恢复和复盘。

