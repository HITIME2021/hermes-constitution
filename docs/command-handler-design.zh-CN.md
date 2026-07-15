# Command Handler 设计

Command Handler 将用户的 slash command 转换成安全、可审计的控制面操作。

Command 不是工作流任务。它不执行 Provider，不直接修改 Task 状态。它是会话/控制面操作，必须声明允许效果、禁止效果，并返回结构化输出。

## 设计原则

```text
command -> command policy gate -> handler -> effects -> result
```

一个 command handler 必须：

- 用 `effects` 声明它允许做什么
- 用 `no_effects` 声明它绝对不能做什么
- 接收结构化输入
- 返回结构化输出
- 不静默跳过已声明的必要效果
- 不执行未声明的副作用
- 不绕过 Project Policy、approval 要求或文件系统边界

Command 不走普通项目任务的 Workflow Engine、Task Planner、Risk Evaluator 或 Review Gate 路径。但它仍然必须先经过 Command Policy Gate。

## Command Policy Gate

Command Policy Gate 在分发 command 前检查：

- command alias 是否只解析到一个已注册 handler
- 参数是否符合 command schema
- 请求的 effects 是否被该 command 允许
- caller context 是否试图附加 forbidden effects
- 必要 approval 是否存在
- 文件写入是否在 command 声明范围内
- 除非明确声明并批准，否则禁止修改目标项目
- 除非明确声明并批准，否则禁止写入长期记忆

如果 gate 无法证明 command 安全，就返回结构化 command error，不分发 handler。

## Command Schema

```yaml
commands:
  <command_name>:
    aliases:
      - /<slash-alias>
    description: "<human-readable description>"
    owner: hermes_core
    parameters:
      <param_name>:
        type: <string|path|boolean|choice>
        default: <default_value>
        required: <true|false>
    effects:
      - <allowed_effect>
    no_effects:
      - <forbidden_effect>
    output:
      required_fields:
        - <field_name>
    errors:
      - <error_type>: <human-readable_message>
```

## Effect 目录

`effects` 表示 command 允许执行的动作。

`no_effects` 表示即使上下文看起来需要，command 也绝对不能执行的动作。

| Effect | 含义 |
|--------|------|
| `read_snapshot_only` | 只读取快照文件，不触碰宪法源仓 |
| `read_full_constitution` | 读取宪法 docs、schemas、decisions |
| `generate_snapshot` | 从源文档生成新的宪法快照 |
| `overwrite_current_snapshot` | 覆盖 `~/hermes-snapshots/current.md` |
| `archive_versioned_snapshot` | 在 `~/hermes-snapshots/archive/` 下写入归档副本 |
| `write_snapshot_index` | 写入 `~/hermes-snapshots/current.index.json`，用于 block 变更追踪 |
| `read_file` | 读取一个或多个文件 |
| `write_file` | 写入或覆盖一个已声明文件 |
| `archive_file` | 将一个已声明文件移动或复制到归档路径 |
| `require_approval` | 执行前要求操作者批准 |

| No Effect | 含义 |
|-----------|------|
| `full_constitution_reload` | 不得遍历或重读完整宪法仓 |
| `task_execution` | 不得调用 Workflow Engine 执行任务 |
| `provider_execution` | 不得调用 Codex、CodeBuddy 或任何 Provider Adapter 执行任务 |
| `project_mutation` | 不得修改任何项目仓 |
| `target_project_mutation` | 不得修改当前目标项目 |
| `source_repo_mutation` | 不得修改宪法源仓 |
| `memory_writeback` | 不得写入长期 Memory Center |
| `provider_routing` | 不得把工作路由给 Provider |
| `task_state_transition` | 不得改变 Task 状态 |
| `delete_file` | 不得删除文件 |

## Command Registry

Registry 是由 Hermes Core 拥有的扁平命名空间。

```text
CommandRegistry
  -> register(name, handler_metadata, handler)
  -> resolve(alias) -> handler
  -> list() -> [command_metadata]
```

默认只有 Hermes Core 可以注册内置 command。

Skill 提供的 command 必须通过 Command Policy 明确注册，并在 metadata 中声明 owning skill。

Provider Adapter 默认不得注册面向用户的 slash command。只有 Project Policy 明确授权，且不会绕过 adapter scope、review 或 approval 边界时，才允许 Provider Adapter command。

## 宪法命令 (v0.1)

### `load_constitution_snapshot`

加载当前会话使用的固定宪法快照。

```yaml
commands:
  load_constitution_snapshot:
    aliases:
      - /load-constitution-snapshot
    description: "Load the pinned constitution snapshot into the current session."
    owner: hermes_core
    parameters:
      path:
        type: path
        default: ~/hermes-snapshots/current.md
        required: false
    effects:
      - read_snapshot_only
    no_effects:
      - full_constitution_reload
      - task_execution
      - provider_execution
      - project_mutation
      - memory_writeback
      - provider_routing
      - task_state_transition
    output:
      required_fields:
        - constitution_version
        - snapshot_path
        - loaded_at
    errors:
      - snapshot_not_found: "Snapshot file does not exist. Run /reload-constitution first."
      - snapshot_version_mismatch: "Snapshot version does not match requested version."
      - permission_denied: "Cannot read the snapshot file."
```

除非快照缺失或过期，且用户明确授权 `/reload-constitution`，否则 handler 不得读取 `~/projects/production/hermes-constitution`。

### `reload_constitution`

从源仓全量重新加载宪法，并重新生成快照。

```yaml
commands:
  reload_constitution:
    aliases:
      - /reload-constitution
    description: "Reload the full constitution and regenerate the current snapshot."
    owner: hermes_core
    parameters:
      source_repo:
        type: path
        default: ~/projects/production/hermes-constitution
        required: false
      current_snapshot:
        type: path
        default: ~/hermes-snapshots/current.md
        required: false
      archive_dir:
        type: path
        default: ~/hermes-snapshots/archive
        required: false
      index_path:
        type: path
        default: ~/hermes-snapshots/current.index.json
        required: false
    effects:
      - read_full_constitution
      - generate_snapshot
      - overwrite_current_snapshot
      - archive_versioned_snapshot
      - write_snapshot_index
    no_effects:
      - task_execution
      - provider_execution
      - target_project_mutation
      - source_repo_mutation
      - memory_writeback
      - provider_routing
      - task_state_transition
      - delete_file
    output:
      required_fields:
        - constitution_version
        - commit
        - snapshot_path
        - archive_path
        - index_path
        - loaded_at
        - added_blocks
        - changed_blocks
        - removed_blocks
    errors:
      - source_repo_not_found: "Constitution repository not found at the expected path."
      - git_head_unavailable: "Cannot determine current git HEAD."
      - snapshot_block_duplicate: "Two or more source blocks declare the same id."
      - snapshot_block_malformed: "A source block marker is malformed."
      - snapshot_write_failed: "Cannot write the generated snapshot."
      - index_write_failed: "Cannot write the generated snapshot index."
      - archive_write_failed: "Cannot write the archived snapshot."
```

该 handler 可以写入 `~/hermes-snapshots/` 下的快照文件。它不得把生成快照写入 `~/projects/production/hermes-constitution`。

`reload_constitution` 必须由源文档驱动：

- 扫描源文档中显式声明的 `snapshot:block` 标记
- 从抽取出的 block set 渲染 `current.md`
- 写入 `current.index.json`，记录 block id、section、priority、source path 与内容 hash
- 对比上一次 index，并报告 added / changed / removed blocks
- 拒绝重复 block id 和格式错误的 block marker
- 不得把旧的 `current.md` 当成策略事实源
- 不得只依赖手工维护的大型模板作为策略内容来源

## Caller Contract

调用 command handler 的一方必须：

- 按声明传入参数
- 尊重 `effects` 和 `no_effects`
- 处理已声明错误类型
- 不绕过 handler 直接调用内部函数
- 不给 `no_effects` 已禁止的 command 叠加 task execution、provider routing、memory writeback 或文件修改

封装 command 的 gateway、CLI 或 chat surface 不得静默添加 command contract 之外的行为。

## 测试

Command handler 应独立测试。

测试契约：

- 每个声明的 effect 都可验证
- 每个声明的 error 都可触发
- 每个 `no_effect` 都被强制执行
- 输出 schema 符合 `required_fields`
- Command Policy Gate 会拒绝未声明或冲突的 effects
- 除非明确声明，否则 command execution 不修改 Task 状态
