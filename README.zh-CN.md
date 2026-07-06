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

## 运行时原则

Hermes Core 默认 headless，不依赖 GUI。

```text
Hermes Core
  负责 workflow、policy、state、memory、routing。

CLI / API
  负责自动化调用 Codex、CodeBuddy 和未来 Provider。

GUI / Desktop
  只是人类驾驶舱，用于观察、调试、审批和协作。
```

所以：

```text
生产自动化走 CLI/API。
人工协作和复杂讨论可以用桌面软件。
```

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
- [运行时与 Provider 接口](docs/runtime-and-provider-interfaces.zh-CN.md)
- [宪法快照策略](docs/constitution-snapshot.zh-CN.md)
- [Command Handler 设计](docs/command-handler-design.zh-CN.md)
- [Provider 认证策略](docs/provider-auth-policy.zh-CN.md)
- [人工介入策略](docs/human-intervention-policy.zh-CN.md)
- [简单 Shell 直通模式](docs/simple-shell-direct-mode.zh-CN.md)
- [项目策略](docs/project-policy.zh-CN.md)
- [工作流状态机](docs/workflow-state-machine.zh-CN.md)
- [能力解析器](docs/capability-resolver.zh-CN.md)
- [Context Manager](docs/context-manager.zh-CN.md)

英文文档仍然作为协议源文档。中文文档用于解释设计意图、边界和取舍。

## 关键决策

- 角色来自 `msitarzewski/agency-agents`，但 Hermes 不直接调度原始 Markdown，而是通过 Adapter 转成 Hermes `AgentProfile`。
- `garrytan/gstack` 可以作为 workflow、skill、review routing 的参考，但不直接作为 Hermes 的评分核心。
- Hermes Core 默认 headless，不依赖 GUI。
- Codex 和 CodeBuddy 的自动化调用优先走 CLI/API。
- GUI / Desktop 只是人类驾驶舱，用于观察、调试、审批和协作。
- Project Policy 优先级高于 AgentProfile、Skill 和 Provider 偏好。
- CodeBuddy 默认禁止修改 `auth`、`security`、`db`、`infra`。
- 新增依赖默认需要 approval。
- medium 及以上风险任务必须有 Codex planning 或 Codex review。
- DeepSeek-V4-Pro 可以作为 LLM model backend 使用，但在 v0.1 中不是正式 Hermes Provider。它不能默认替代 Codex 的 planning/review，也不能替代 CodeBuddy 的 scoped execution，除非 Project Policy 明确升级它。
- Codex CLI 默认使用操作者在 WSL 中通过 `codex login` 建立的 ChatGPT/OAuth 会话。Hermes 不得默认注入 `OPENAI_API_KEY`，也不得查看或保存认证材料。
- 普通会话默认加载 `~/hermes-snapshots/current.md` 作为 constitution snapshot，不应每轮全量读取 `~/projects/hermes-constitution`。
- `/reload-constitution` 应从源文档声明的 `snapshot:block` 标记生成快照，并写入 `current.index.json`；不得把手工维护的大型模板当作策略事实源。
- 用户 slash command 应通过 Command Handler 实现，并声明 `effects`、`no_effects`，先经过 Command Policy Gate。
- 安全、高频、只读的 shell 检查命令可以使用简单直通模式，减少不必要的规划和 token 消耗。
- 自动化重试、修正和重规划都有预算；重复失败、方向不清、范围不安全或 token/time 不值得时，必须停止并请求人工介入。

## WSL 如何使用

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
4. docs/runtime-and-provider-interfaces.md
5. docs/constitution-snapshot.md
6. docs/command-handler-design.md
7. docs/provider-auth-policy.md
8. docs/human-intervention-policy.md
9. docs/simple-shell-direct-mode.md
10. docs/project-policy.md
11. docs/workflow-state-machine.md
12. docs/capability-resolver.md
13. docs/context-manager.md
14. docs/execution-protocol.md
15. docs/review-gate.md
16. docs/memory-center.md
17. docs/agent-profile-and-skills.md
18. schemas/*.yaml
19. decisions/*.md
```

## WSL / Windows 运行面原则

Hermes 在你的 Windows 11 + WSL 环境中默认遵守：

```text
WSL = production execution plane
Windows = human control / constitution maintenance plane
```

Codex CLI、CodeBuddy CLI、Hermes Core、Provider Adapter、Python/uv、Node.js、测试、lint、build 和包管理器都应放在 WSL。Windows 只作为 Codex Desktop / GUI、人工批准、宪法维护、策略审查、交互调试和观察的控制面。

Hermes 不应把同一个执行任务拆到 Windows 和 WSL 两边完成。实现、测试、依赖、Provider 执行和自动化状态默认在 WSL 执行；宪法设计、策略审查、批准和人类协作可以在 Windows 控制面完成。

## 人类输出与记忆语言原则

Hermes 在这个操作者环境中，面向用户的输出和长期记忆默认使用简体中文，方便后续人工审查、筛选和清理记忆体。

默认使用中文的内容包括：

- 任务摘要
- 规划解释
- 风险说明
- Review 结论
- Memory candidate
- Memory claim / summary
- 人工批准问题
- 记忆清理备注

但以下内容必须保持原样，不翻译、不改写：

- JSON / YAML 字段名
- schema id
- 文件路径
- shell 命令
- 代码符号
- 包名
- Provider 名称
- 引用的原文证据

英文宪法文档仍然是协议事实源；中文内容用于操作者解释、偏好和人工维护。由英文协议生成长期记忆时，Hermes 应写入一条中文 claim，并保留英文来源作为 evidence。

建议先让 Hermes agent 做一次 dry-run，不要马上接真实 CodeBuddy：

```text
读取 constitution
输出它理解到的 v0.1 定位、Provider 分工、Project Policy、Risk Policy、Execution Protocol
然后模拟一个低风险任务，生成 Task、RiskEvaluation、ExecutionPlan、ExecutionRequest、ReviewPlan、MemoryCandidate
```
