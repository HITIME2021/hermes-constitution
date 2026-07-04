# 宪法快照策略

宪法快照是当前 Hermes 宪法的压缩版中文摘要，用于减少每次会话重复读取完整 docs/schema 的上下文成本。

完整 `hermes-constitution` 仓库仍然是事实源。快照只是绑定到某个 constitution commit 的本地缓存。

## 存储策略

生成的快照不得写入 `hermes-constitution` 仓库。

默认本地存储位置：

```yaml
constitution_snapshot:
  default_path: ~/hermes-snapshots/current.md
  archive_path_template: ~/hermes-snapshots/archive/constitution-<commit>-<date>.md
  source_repo: ~/projects/hermes-constitution
```

`current.md` 是稳定加载入口。重新生成快照时可以覆盖它。

`archive/constitution-<commit>-<date>.md` 是可选历史归档，用于人工审查和回滚。

## 默认行为

普通会话默认加载快照，不全量读取宪法仓。

```text
默认启动：
  /load-constitution-snapshot ~/hermes-snapshots/current.md
```

当策略会影响任务行为时，Hermes 应在输出中标注 `constitution_version`。

## 重新加载条件

只有以下情况才应全量重新加载宪法：

- 用户明确要求 `reload constitution` 或 `/reload-constitution`
- 源仓库 git `HEAD` 变化
- 快照文件缺失
- 快照版本与请求的 constitution version 不匹配
- schema 校验失败
- 发现 policy 冲突
- Hermes 行为与已加载快照不一致

## 命令定义

### `/load-constitution-snapshot [path]`

把指定快照加载为当前会话宪法缓存。

效果：

- 只读取快照文件
- 不读取完整 `~/projects/hermes-constitution` 仓库
- 输出 `constitution_version`、`snapshot_path` 和 `loaded_at`
- 不执行项目任务
- 不修改项目文件
- 不写入长期记忆

默认路径：

```text
~/hermes-snapshots/current.md
```

### `/reload-constitution`

全量读取宪法并重新生成快照。

效果：

- 读取 `~/projects/hermes-constitution`
- 获取当前 git `HEAD`
- 生成简体中文 constitution snapshot
- 覆盖 `~/hermes-snapshots/current.md`
- 归档一份到 `~/hermes-snapshots/archive/constitution-<commit>-<date>.md`
- 输出新的 `constitution_version`
- 不执行项目任务
- 不修改目标项目
- 除非用户明确批准，否则不写入长期记忆

## 快照内容与格式

有效快照应包含：

- constitution version / git commit
- 运行面策略
- provider roles
- model backends
- 语言策略
- dry-run 策略
- memory 策略
- 人工介入策略
- 简单 shell 直通模式
- risk / review 策略
- provider adapter 边界
- reload 条件

快照应优先保证操作者可审查性，而不是追求极限压缩。目标是减少重复加载完整宪法，同时让人类容易审查、比较和清理。

允许使用：

- 简洁表格
- 矩阵式块
- 短分组列表
- 稳定章节编号
- 针对 command 或 execution 规则的简短示例

以下内容优先使用更清晰的表格或矩阵结构：

- provider roles
- model backends
- WSL / Windows 执行面
- 语言策略
- dry-run 与 memory 策略
- 人工介入预算与停止条件
- command handler 的 effects / no_effects
- 简单 shell 直通模式白名单

默认长度目标为 120-180 行。更短并不必然更好；如果压缩导致人工审查、比较或清理更困难，应优先保留清晰结构。
