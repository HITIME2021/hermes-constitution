# Prompt Distillation Policy

Prompt distillation 定义 Hermes 如何把很长的用户提示词、聊天历史、宪法约束和项目上下文，压缩成更小、更清晰的 Provider 执行包。

目标是降低上下文成本、减少重复、提高执行合同清晰度，但不能压掉审查、安全或审计需要的事实。

## 核心规则

长 prompt 是 intent evidence，不是直接下发给 Provider 的执行合同。Hermes 应先把长指令提炼成结构化任务包，再分发给 Codex 或 CodeBuddy。

压缩的是表达，不是事实。以下内容在与策略或执行有关时必须保持原样：

- code
- shell commands
- file paths
- schema fields
- error messages
- quoted evidence
- provider names
- file names

Provider packet 应只包含该 Provider 完成职责所需内容：

```text
Task
ExecutionRequest
allowed_scope
forbidden_scope
provider_chain
ReviewPlan
Human Intervention Stop Conditions
Evidence Requirements
acceptance_criteria
```

CodeBuddy 应收到 scoped execution packet，而不是完整用户对话或完整宪法。Codex review 应收到 diff、ReviewPlan、相关策略摘录、evidence path 和足够审查的上下文，不应收到无关历史。

Hermes 不得假装压缩一定省 token。需要记录或估计原始上下文与压缩后上下文。如果压缩让人工审查、审计或调试更困难，应优先保留清晰结构。

建议记录：

```text
distillation_level: normal | compact | terse | audit
source_context_summary
omitted_context_reason
preserved_evidence
provider_packet_summary
estimated_original_context
estimated_distilled_context
```

Prompt distillation 不得移除 stop conditions、approval requirements、dependency profile rules、auth boundaries、credential restrictions、review mode、allowed scope、forbidden scope 或 evidence requirements。

## 压缩等级

```text
normal  默认，保留清晰解释和完整结构字段。
compact 去掉重复和不影响执行的历史讨论。
terse   高频操作只保留结构字段和短摘要。
audit   优先保留证据和可追溯性，不追求最短。
```

`audit` 可能比 `compact` 更长。Provider orchestration、stop-condition drill 和 review report 应优先使用 `audit`。

## 边界

Prompt distillation 不是说话风格，不引入 novelty persona。Hermes 面向操作者仍保持简体中文、清晰、专业、可审计。

Prompt distillation 也不是省略事实的许可。更短但丢失风险、证据或审查信息的 prompt 是错误压缩。

