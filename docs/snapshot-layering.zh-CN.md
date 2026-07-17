# Snapshot Layering 策略（中文说明）

本文是 `docs/snapshot-layering.md` 的中文说明。英文文件中的
`snapshot:block` 是生成快照时的规范性来源。

## 目标

随着 Hermes constitution 继续增长，单一 `current.md` 会越来越长。
v0.4.1 的目标不是删除策略内容，而是把生成快照拆成可加载层：

```text
boot.md       -> 启动可信验证、版本、index、加载顺序、fallback
core.md       -> 每次正常工作都需要的核心运行策略
packs/*.md    -> 按任务领域加载的策略包
current.md    -> 完整兼容快照，保留给旧 loader 和恢复路径
```

完整 `hermes-constitution` 源仓仍然是事实源。所有快照层都是
`~/hermes-snapshots/` 下的生成缓存。

## boot.md

`boot.md` 只负责可信启动，不是执行授权。

它应包含：

- `constitution_version`
- human-facing `release`
- `source_repo`
- `current.index.json` 路径和 hash
- `core.md` 路径
- `packs/` 路径
- `current.md` 兼容 fallback
- 加载顺序

如果 `boot.md` 或 `current.index.json` 缺失、不可读、格式错误或 hash
不一致，Hermes 必须在任何工具调用、provider 调用、shell、memory
写入、自改进或任务执行前停止，返回 `UNTRUSTED_CONTEXT_STOP`。

## core.md

`core.md` 是正常工作默认加载的核心策略层，应该覆盖：

- constitution version 归因
- session startup / reload 条件
- gateway entry guard / stale session invalidation
- workspace layout
- command handler effect model
- project policy 优先级
- provider adapter 边界和默认禁止 self-edit
- planning / execution / review / stop-condition gates
- memory writeback approval boundary
- token + quality telemetry
- simple shell direct mode 安全摘要
- local background model authority boundary

## domain packs

`packs/*.md` 是按需加载的领域策略包。初始建议：

| Pack | 触发场景 |
|------|----------|
| `commands.md` | slash command、reload/load snapshot、command effects |
| `gateway.md` | gateway、DM、mobile、webhook、pairing、入口可信 |
| `local-models.md` | Ollama、本地模型、context budget |
| `memory.md` | MemoryCandidate、writeback、memory synthesis |
| `project-policy.md` | dependency、auth、security、db、infra、目标仓 scope |
| `provider-adapters.md` | Codex、CodeBuddy、Hermes Primary、provider confirmation |
| `review-and-execution.md` | dry-run、ExecutionRequest、stop conditions、Codex review |
| `tools-and-artifacts.md` | Tools Layer、Tools Adapter、Artifact Intake Gate |
| `workspace.md` | production/labs/worktrees/archive layout |

Pack 选择必须是确定性和可审计的。模型可以建议 pack set，但不能成为
最终权威；Hermes 必须根据命令类型、入口、任务分类、declared effects 和
operator 显式请求计算最终 pack set。

## current.md 兼容

`current.md` 不能删除。它仍然是完整兼容快照：

- 旧 `/load-constitution-snapshot ~/hermes-snapshots/current.md` 继续有效
- gateway guard 可以继续验证 `current.md + current.index.json`
- `current.md` 必须和 layered files 来自同一个 block set
- `current.md` 不能变成手工维护模板

## 实施顺序

1. 先更新 constitution 文档和 index metadata 设计。
2. 扩展 `/reload-constitution`，在保留 `current.md` 的同时输出
   `boot.md`、`core.md`、`packs/*.md`。
3. 扩展 `/load-constitution-snapshot`，支持 `current_compat`、
   `boot_core`、`boot_core_domain`。
4. 给 gateway/DM startup 增加 `boot.md + index` 验证。
5. 增加 deterministic pack selection 和测试。
6. 验证完成后再把 human-facing release 标记为 `v0.4.1`。
