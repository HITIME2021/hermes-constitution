# Hermes Constitution 中文说明

本仓库保存 Hermes v0.1 的工作宪法。

这里的“宪法”不是最终产品文档，而是 Hermes agent 在启动、规划、派发任务、审查结果和写入记忆时应当遵守的基础规则。

## Hermes 是什么

Hermes 是一个多 Agent 自动化调度平台。

它的核心职责不是亲自写完所有代码，而是：

```text
理解用户目标
拆解任务
选择角色
选择技能
选择执行 Provider
控制风险和成本
审查结果
沉淀经验
```

当前 v0.1 的基本分工是：

```text
Codex / GPT-5.5
  高级大脑，负责架构、规划、算法、风险判断、review、replanning、memory synthesis。

CodeBuddy
  低成本执行者，负责明确边界内的代码修改、重复性实现、简单 bugfix、补测试。

Claude Code
  v0.1 暂时排除。
```

## 核心流程

```text
User Intent
  -> Intake / Clarification
  -> Workflow Engine
  -> Task Planner
  -> Risk Evaluator
  -> Agency Registry
  -> Capability Resolver
  -> Provider Routing
  -> Execution
  -> Review Gate
  -> Testing / Verification
  -> Finalize
  -> Memory Writeback
```

最重要的调度链路是：

```text
task -> role -> skill -> provider -> review -> memory
```

Hermes 不应该简单地把任务交给某个 Agent，而应该先判断任务类型、风险、角色、技能、上下文、Provider 能力和审查等级。

## 文档结构

```text
docs/
  架构和模块设计文档。

schemas/
  Hermes 核心对象的 YAML schema 草案。保持英文，作为机器协议源。

decisions/
  架构决策记录。

diagrams/
  Mermaid 流程图。
```

## 中文文档

优先阅读：

- [Hermes v0.1 架构](docs/hermes-v0.1-architecture.zh-CN.md)
- [项目策略](docs/project-policy.zh-CN.md)
- [工作流状态机](docs/workflow-state-machine.zh-CN.md)
- [能力解析器](docs/capability-resolver.zh-CN.md)
- [Context Manager](docs/context-manager.zh-CN.md)

英文文档仍然作为协议源文档。中文文档用于解释设计意图、边界和取舍。

## 关键决策

- 角色来自 `msitarzewski/agency-agents`，但 Hermes 不直接调度原始 Markdown，而是通过 Adapter 转成 Hermes `AgentProfile`。
- `garrytan/gstack` 可以作为 workflow、skill、review routing 的参考，但不直接作为 Hermes 的评分核心。
- Project Policy 优先级高于 AgentProfile、Skill 和 Provider 偏好。
- CodeBuddy 默认禁止修改 `auth`、`security`、`db`、`infra`。
- 新增依赖默认需要 approval。
- medium 及以上风险任务必须有 Codex planning 或 Codex review。

## 回家后如何使用

在运行 Hermes agent 的 Windows 11 + WSL 机器上：

```bash
cd ~/projects
git clone https://github.com/HITIME2021/hermes-constitution.git
cd hermes-constitution
```

然后让 Hermes agent 按固定顺序读取：

```text
1. README.md
2. README.zh-CN.md
3. docs/hermes-v0.1-architecture.md
4. docs/project-policy.md
5. docs/workflow-state-machine.md
6. docs/capability-resolver.md
7. docs/context-manager.md
8. docs/execution-protocol.md
9. docs/review-gate.md
10. docs/memory-center.md
11. docs/agent-profile-and-skills.md
12. schemas/*.yaml
13. decisions/*.md
```

建议先让 Hermes agent 做一次 dry-run，不要马上接真实 CodeBuddy：

```text
读取 constitution
输出它理解到的 v0.1 定位、Provider 分工、Project Policy、Risk Policy、Execution Protocol
然后模拟一个低风险任务，生成 Task、RiskEvaluation、ExecutionPlan、ExecutionRequest、ReviewPlan、MemoryCandidate
```

