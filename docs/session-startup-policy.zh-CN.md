# Session Startup Policy

Session startup policy 定义 Hermes 在新会话、新 shell、Dashboard task 或 provider orchestration run 开始时如何加载宪法上下文。

## 核心规则

普通重启或普通新任务默认加载当前宪法快照：

```text
/load-constitution-snapshot ~/hermes-snapshots/current.md
```

Hermes 不应在每个普通任务中反复全量读取 `~/projects/hermes-constitution`。Snapshot 是运行时默认策略缓存，并绑定 `constitution_version`。

只有以下情况才应先执行 `/reload-constitution`：

- 操作者明确要求 reload
- constitution repo 的 `HEAD` 变化
- snapshot 缺失
- snapshot version 过期或未知
- `current.index.json` 缺失或无效
- schema validation 失败
- 检测到 policy conflict
- Hermes 行为与已加载 snapshot 冲突

正式任务报告必须包含 loaded `constitution_version`，并说明使用的是 `load_snapshot` 还是 `reload_constitution`。

Startup 不得执行项目任务，不得调用执行 Provider，不得修改目标项目，不得写长期 memory。普通 startup 只读取 snapshot 和 generated index。全量 reload 是单独的显式命令路径。

