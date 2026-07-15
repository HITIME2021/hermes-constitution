# Workspace Layout Policy 工作区布局策略

Workspace Layout Policy 定义 Hermes 在操作者 WSL 环境中应当把生产仓库、工具实验、
smoke 项目、worktree 和归档测试产物放在哪里。

## 核心规则

<!-- snapshot:block id="workspace-layout-policy.zh-CN" section="Workspace Layout" priority="58" -->
Hermes 必须把生产仓库、实验项目、provider worktree 和归档实验分开放在不同的
workspace 子目录中。

推荐 WSL 布局：

```text
~/projects/
  production/
    GitHub-Trend-Intelligence-Platform/
    hermes-constitution/

  labs/
    spec-kit-smoke-001/
    hermes-kanban-provider-smoke/
    hermes-kanban-comment-contract-smoke/

  worktrees/
    GitHub-Trend-Intelligence-Platform-hermes-run-011/
    ...

  archive/
    ...
```

目录含义：

```text
~/projects/production/  -> 真实长期维护仓库
~/projects/labs/        -> smoke test、工具验证、一次性实验
~/projects/worktrees/   -> git worktree 和 Hermes run 工作区
~/projects/archive/     -> 已废弃但暂时保留以供检查的旧实验
```

Hermes 不得直接在 `~/projects` 下创建 smoke、lab 或 tool-validation 项目。
新的实验项目应放在 `~/projects/labs`。Provider orchestration worktree 应放在
`~/projects/worktrees`，除非操作者明确选择其他路径。

生产仓库不得作为 frontend tool 实验的 scratch space，例如 spec generation、
scaffold trial 或 workflow validation。可能写文件的工具，例如 project initializer
或 template installer，应先在 `~/projects/labs/<tool-smoke-name>` 或明确声明的隔离
workspace 中测试。

Hermes 必须在 run evidence 中记录 workspace class：

```yaml
workspace:
  root: ~/projects/labs/spec-kit-smoke-001
  class: lab
  purpose: tool_validation
  target_project: none
```

如果任务目标是生产仓库，Hermes 必须显式声明，不得仅凭目录名推断 production status。
<!-- /snapshot:block -->

## 非目标

本策略不要求立即迁移现有仓库。

本策略不覆盖操作者对具体任务明确批准的路径。

本策略不替代 `allowed_scope`、`forbidden_scope`、worktree evidence 或 stop-condition
检查。
