# Constitution Release

本文档记录 Hermes 宪法的人类可读 release line。

它不同于：

- JSON/YAML schema version，例如 `0.1.0`
- 目标项目版本
- Provider CLI 版本
- snapshot 中基于 git commit 的 `constitution_version`

## 当前版本

<!-- snapshot:block id="constitution-release.zh-CN" section="Constitution Release" priority="11" -->
当前 Hermes 宪法 release：`v0.3.2`。

`v0.3.2` 表示宪法在已验证的 v0.2 本地编排基线上，新增了工具治理、artifact intake、
planning source control、token/quality telemetry、Simple Shell Direct Mode 安全边界，
以及更严格的 Hermes self-edit 边界。它还新增 WSL workspace layout separation，
用于区分生产仓库、实验项目、provider worktree 和归档实验。它进一步新增 Gateway
Entry Guard 和 Self-Improvement Governance，使 DM/mobile 入口只有在 startup
verification 后才可信，并让 self-improvement 在写入前先生成 candidate。

已验证的 v0.2 控制回路：

- source-driven constitution snapshot，包含 block extraction 与 index output
- Codex planning + CodeBuddy scoped execution + Codex review
- 基于目标项目 dependency profile 的依赖策略
- low-risk production-code bugfix orchestration
- Dashboard / Kanban / Gateway 对 Hermes-managed task 的生命周期可视化
- provider orchestration observability policy

v0.3 新增能力：

- Tools Layer 分类：Frontend Tools produce artifacts；Backend Tools produce effects；Hermes governs both
- `planning_source_of_record`：`codex_native` 或 `frontend_artifact_assisted`
- Frontend artifacts 影响 live execution 前必须经过 Artifact Intake Gate
- Token telemetry 必须与 quality telemetry 配对
- Simple Shell Direct Mode 进入 snapshot，并采用 hard-reject safety posture
- Hermes self-edit 默认禁用；implementation work 默认路由到 CodeBuddy scoped execution、verification 和 Codex review，除非操作者对具体任务明确给出 emergency override
- Workspace Layout Policy 区分 `~/projects/production`、`~/projects/labs`、`~/projects/worktrees` 和 `~/projects/archive`
- Gateway Entry Guard 要求非 TUI 入口在执行工具前验证 `current.md` 和 `current.index.json`
- Self-Improvement Governance 允许自动生成 candidate，但 apply patch 是 authority-bearing effect

已加载 snapshot 的精确 `constitution_version` 仍然是 git commit。release label 是面向人的成熟度标记。
<!-- /snapshot:block -->

## 版本含义

`v0.1` 是架构与策略基线。

`v0.2` 是当前操作者环境中的本地编排验证基线：

```text
WSL = production execution plane
Windows = human control and constitution maintenance plane
Codex = planning and review
CodeBuddy = scoped execution
Dashboard/Kanban = observability for Hermes-managed runs
```

`v0.3` 是工具治理与 artifact intake 基线：

```text
Frontend Tools = artifact evidence, not authority
Backend Tools = authority-bearing effects with scope control
Codex native planning = default planning mode
Frontend artifact assistance = optional for complex or ambiguous work
Hermes ExecutionRequest = live execution authority
Hermes self-edit = disabled by default
```

`v0.3.1` 是 v0.3 线的 workspace layout patch：

```text
production repos -> ~/projects/production
tool validation and smoke projects -> ~/projects/labs
provider worktrees -> ~/projects/worktrees
retained old experiments -> ~/projects/archive
```

`v0.3.2` 是 trusted-entry 与 self-improvement governance patch：

```text
gateway / DM / mobile entrypoints -> untrusted until startup verification
current.md + current.index.json -> required before tool execution
self-improvement analysis -> allowed
self-improvement patch application -> trusted channel + scope + approval
```

未来版本号提升应通过 ADR 记录，并说明验证了什么能力，而不只是说明文档有变更。
