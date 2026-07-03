# Context Manager 中文说明

Context Manager 负责把“全量项目现实”裁剪成“当前 Provider 完成当前任务所需的最小充分上下文”。

核心原则：

```text
Codex 看全局。
CodeBuddy 看局部。
```

更准确地说：

```text
Codex Context = 目标 + 架构 + 风险 + 约束 + 相关代码 + 历史经验
CodeBuddy Context = 目标 + 指定文件 + 明确步骤 + 验收标准 + 禁止事项
```

## 职责

```text
收集上下文
选择上下文
压缩上下文
分发上下文
隔离上下文
记录上下文使用反馈
```

Context Manager 不直接决定 Provider，它为 Capability Resolver 提供 `context_fit` 和 `context_cost`。

## Context Scope

上下文范围分级：

```text
none
file
file_set
module
repo_map
full_project
```

默认映射：

```text
trivial -> file 或 file_set
low -> file_set
medium -> file_set 或 module
high -> module + repo_map
critical -> repo_map + targeted files + Codex reasoning context
```

## 给 Codex 的上下文

Codex 用于判断、规划、review、replanning，因此可以看到更完整的上下文：

```text
用户目标
验收标准
项目结构摘要
技术栈
相关文件
项目约定
风险初判
Memory hints
Provider options
```

## 给 CodeBuddy 的上下文

CodeBuddy 拿到的是执行包，而不是完整 Hermes 内部推理：

```text
task_id
role
skill
goal
allowed_files
forbidden_files
constraints
acceptance_criteria
context_snippets
recommended_tests
report_contract
```

CodeBuddy 不需要知道完整 repo mental model，也不应该看到未授权文件内容。

## 不可压缩字段

上下文预算不够时可以压缩代码、摘要、历史经验，但以下内容不能被压缩掉：

```text
验收标准
allowed scope
forbidden scope
风险控制
approval 要求
report contract
```

## 安全隔离

硬规则：

```text
Secrets、tokens、credentials 永远不进入 Provider 上下文。
CodeBuddy 不接收完整 Hermes internal reasoning。
禁止文件可以给路径，但不要给内容。
用户长期个人记忆只在必要时给 Codex，不给 CodeBuddy。
Memory hints 必须被筛选，不把原始历史日志塞给执行器。
```

## 与 Memory 的关系

```text
Memory Center 提供候选记忆。
Context Manager 决定哪些记忆进入本次上下文。
```

Memory 不应该直接喂给 Provider。

## Context Feedback

每次任务后，Context Manager 应记录：

```text
使用了哪些文件
哪些上下文缺失
哪些上下文没用
Provider 是否因为上下文不足失败
当前 context_scope 是否合适
```

这些反馈会反哺 Memory Center 和 Capability Resolver。

