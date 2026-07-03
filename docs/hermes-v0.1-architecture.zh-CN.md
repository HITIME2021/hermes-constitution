# Hermes v0.1 架构说明

## 定义

Hermes 是一个多 Agent 自动化调度平台。它接收用户目标，将目标拆解成结构化任务，选择角色和技能，路由到合适的执行 Provider，审查结果，并把可复用经验写入长期记忆。

Hermes 不是单个 Agent，而是 Agent 和执行 Provider 之上的操作层。

## v0.1 范围

v0.1 包含：

```text
Workflow Engine
Task Planner
Risk Evaluator
Agency Registry
Capability Resolver
HAS Skill Registry
Context Manager
Memory Center
Project Policy
Review Gate
Execution Protocol
Codex Provider
CodeBuddy Provider
```

v0.1 暂时不包含：

```text
Claude Code provider
生产部署自动化
组织级长期记忆
critical 操作的完全自动执行
```

## 分层架构

```text
Hermes
|
+-- Workflow Layer
|   +-- Workflow Engine
|   +-- State Manager
|   +-- Static Templates
|   +-- Dynamic Replanner
|
+-- Intelligence Layer
|   +-- Codex Brain
|   +-- Task Planner
|   +-- Risk Evaluator
|   +-- Reviewer
|
+-- Agency Layer
|   +-- Agency Registry
|   +-- agency-agents Adapter
|   +-- AgentProfile
|
+-- Capability Layer
|   +-- Capability Resolver
|   +-- HAS Skill Registry
|   +-- Provider Matrix
|   +-- Routing Policy
|
+-- Context & Memory Layer
|   +-- Context Manager
|   +-- Memory Center
|   +-- Project Knowledge Base
|
+-- Execution Layer
|   +-- Codex Provider
|   +-- CodeBuddy Provider
|
+-- Review Layer
    +-- Review Gate
    +-- Verifier
    +-- Quality Policy
```

## Provider 分工

### Codex / GPT-5.5

Codex 是高级大脑。

负责：

```text
架构设计
算法设计
任务拆解
风险判断
Provider 路由
代码审查
失败分析
重新规划
记忆总结
```

不优先用于：

```text
大量机械改动
重复样板代码
低价值批量生成
```

### CodeBuddy

CodeBuddy 是低成本执行者。

负责：

```text
明确边界内的代码修改
代码生成
重复性重构
简单 bugfix
补测试
文件级变更
```

不能独立处理：

```text
架构决策
模糊需求
安全敏感逻辑
不可逆操作
生产变更
跨系统设计
```

## 默认运行模式

### stable

默认模式。

```text
Codex Plan
  -> CodeBuddy Execute
  -> Local Verify
  -> Codex Review when required
  -> Memory Writeback
```

这个模式优先保证稳定性，同时控制成本。

### quality

质量优先模式。

```text
Codex Deep Plan
  -> Codex Design Review
  -> CodeBuddy Scoped Execution
  -> Local Verify
  -> Codex Code Review
  -> Codex Risk Review
  -> Memory Writeback
```

适合核心模块、重要发布、高风险变更、复杂架构任务。

## 外部参考

### agency-agents

`msitarzewski/agency-agents` 被视为外部角色库。Hermes 不直接调度原始 Markdown 角色，而是通过 Adapter 转成 Hermes 内部的 `AgentProfile`。

```text
agency-agents Markdown
  -> Agency Adapter
  -> Hermes AgentProfile
```

### gstack

`garrytan/gstack` 适合作为 workflow 和 skill 设计参考，尤其是：

```text
Think -> Plan -> Build -> Review -> Test -> Ship -> Reflect
```

但 gstack 不应直接成为 Hermes 的 Capability Resolver 评分核心。

