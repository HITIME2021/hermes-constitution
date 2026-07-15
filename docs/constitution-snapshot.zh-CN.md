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
  index_path: ~/hermes-snapshots/current.index.json
  source_repo: ~/projects/production/hermes-constitution
```

`current.md` 是稳定加载入口。重新生成快照时可以覆盖它。

`archive/constitution-<commit>-<date>.md` 是可选历史归档，用于人工审查和回滚。

`current.index.json` 记录生成 `current.md` 时使用的 snapshot blocks。它是生成产物，不得写入 `hermes-constitution` 源仓。

## 默认行为

普通会话默认加载快照，不全量读取宪法仓。

```text
默认启动：
  /load-constitution-snapshot ~/hermes-snapshots/current.md
```

当策略会影响任务行为时，Hermes 应在输出中标注 `constitution_version`。

<!-- snapshot:block id="constitution-version-attribution.zh-CN" section="Constitution Snapshot" priority="31" -->
任务报告必须把策略决策归因到已加载的 constitution snapshot version。Hermes 必须从已加载快照或其生成的 index 读取 `constitution_version`，不得从目标项目仓库推断。目标项目缺少 `.hermes/constitution.md` 不代表 constitution version 是 `N/A`。
<!-- /snapshot:block -->

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
- 不读取完整 `~/projects/production/hermes-constitution` 仓库
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

- 读取 `~/projects/production/hermes-constitution`
- 获取当前 git `HEAD`
- 扫描源文档中声明的 snapshot blocks
- 从 block set 生成简体中文 constitution snapshot
- 覆盖 `~/hermes-snapshots/current.md`
- 归档一份到 `~/hermes-snapshots/archive/constitution-<commit>-<date>.md`
- 写入 `~/hermes-snapshots/current.index.json`
- 对比上一次 index，输出 added / changed / removed block id
- 输出新的 `constitution_version`
- 不执行项目任务
- 不修改目标项目
- 除非用户明确批准，否则不写入长期记忆

## 源驱动 Snapshot Blocks

Snapshot 生成必须由源文档驱动。生成器不得把一个大型手工模板当作策略内容的事实源。

源文档可以用显式 block 标记声明必须进入快照的内容：

```md
<!-- snapshot:block-example id="dependency-policy" section="Project Policy" priority="40" -->
依赖变更由目标项目的 dependency profile 判断。
dependency manifest 与 lockfile 变更默认需要 approval。
<!-- /snapshot:block-example -->
```

Block 规则：

- `id` 必须稳定，并且在仓库内唯一
- `section` 用于把相关 block 归入快照章节
- `priority` 控制同一 section 内的排序
- block 正文是策略内容，必须保留其语义
- 英文源文档仍是协议事实源
- 中文 source block 可用于操作者可读的快照措辞

`/reload-constitution` 必须扫描源文档、抽取 snapshot blocks、按 section 与 priority 排序，然后由得到的 block set 渲染 `current.md`。

旧的 `current.md` 不是事实源。生成器可以用上一次 snapshot index 做差异报告，但不得把旧 `current.md` 当成权威文件进行自由文本 patch。

## Snapshot Index

每次 reload 应在 current snapshot 旁边写入 index：

```json
{
  "schema": "hermes.constitution_snapshot_index.v1",
  "constitution_version": "acc73eb",
  "generated_at": "2026-07-06T00:00:00Z",
  "source_repo": "~/projects/production/hermes-constitution",
  "snapshot_path": "~/hermes-snapshots/current.md",
  "blocks": [
    {
      "id": "dependency-policy",
      "section": "Project Policy",
      "priority": 40,
      "source": "docs/project-policy.md",
      "hash": "sha256:..."
    }
  ]
}
```

Index 用于检测并报告：

- `added` blocks
- `changed` blocks
- `removed` blocks
- 重复 block id
- 缺失的必要 section

变更检测不得只靠文件名启发式判断策略。应使用 block id 和内容 hash。

## 快照内容与格式

有效快照应包含：

- constitution version / git commit
- 运行面策略
- provider roles
- provider auth 策略
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
- provider auth 与 surface 分离
- model backends
- WSL / Windows 执行面
- 语言策略
- dry-run 与 memory 策略
- 人工介入预算与停止条件
- command handler 的 effects / no_effects
- 简单 shell 直通模式白名单

快照长度只是软性优化，不是正确性限制。不得为了控制行数而遗漏必要策略。

指导原则：

- 在不损害清晰度的前提下尽量简洁
- 当前关键策略 section 必须完整覆盖
- 明确预算、停止条件、白名单和禁止动作必须保留
- dependency manifest 与 lockfile 的 profile 规则必须保留
- 所有已声明 snapshot blocks 必须保留，除非 block 本身无效
- 遗漏、重复或格式错误的 block 必须作为生成缺陷报告
- 宪法增长后，快照可以超过 180 行
- 如果短快照遗漏策略，应视为缺陷

更短并不必然更好；如果压缩导致信息丢失，或让人工审查、比较、清理更困难，应优先保留清晰结构。
