# Constitution Release

本文档记录 Hermes 宪法的人类可读 release line。

它不同于：

- JSON/YAML schema version，例如 `0.1.0`
- 目标项目版本
- Provider CLI 版本
- snapshot 中基于 git commit 的 `constitution_version`

## 当前版本

<!-- snapshot:block id="constitution-release.zh-CN" section="Constitution Release" priority="11" -->
当前 Hermes 宪法 release：`v0.2`。

`v0.2` 表示宪法已经不再只是 v0.1 的初始设计基线，而是完成了第一批实际控制回路验证：

- source-driven constitution snapshot，包含 block extraction 与 index output
- Codex planning + CodeBuddy scoped execution + Codex review
- 基于目标项目 dependency profile 的依赖策略
- low-risk production-code bugfix orchestration
- Dashboard / Kanban / Gateway 对 Hermes-managed task 的生命周期可视化
- provider orchestration observability policy

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

未来版本号提升应通过 ADR 记录，并说明验证了什么能力，而不只是说明文档有变更。
