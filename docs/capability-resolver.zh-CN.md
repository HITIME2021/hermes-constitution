# Capability Resolver 中文说明

Capability Resolver 是 Hermes 的派单大脑。它决定当前任务需要什么角色、技能、上下文、Provider 和审查等级。

它不应该只输出一个 Provider，而应该输出一个执行组合：

```yaml
planner: codex
executor: codebuddy
reviewer: codex
verifier: local-tools
fallback: codex
```

## 三层结构

```text
1. Hard Filters
2. Scoring
3. Escalation Rules
```

## Hard Filters

硬过滤是不讨论分数的，不能做就是不能做。

例子：

```text
Provider 没有写文件能力 -> 不能执行代码修改
上下文窗口不够 -> 不能处理全项目架构分析
触碰 forbidden path -> 不能派给 CodeBuddy
high / critical 风险 -> 不能让 CodeBuddy 独立处理
涉及 secrets / destructive action -> 必须更强控制
```

先保证不会乱来，再考虑成本。

## Scoring

通过硬过滤之后，才进入软评分。

评分维度：

```text
role_match
capability_match
skill_match
context_fit
reliability
speed
cost
review_coverage
```

stable 模式偏稳定：

```text
能力匹配、可靠性、review coverage 权重更高。
成本有影响，但不压过稳定性。
```

quality 模式偏质量：

```text
Codex 参与更多。
context_fit 和 reliability 权重更高。
cost 权重下降。
```

## Risk Level

风险等级：

```text
trivial
low
medium
high
critical
```

路由策略：

```text
trivial:
  CodeBuddy 执行，轻量 review 或无 review。

low:
  CodeBuddy 执行，boundary review + automated verification。

medium:
  Codex planning + CodeBuddy execution + Codex review。

high:
  Codex planning + scoped execution + Codex review + stronger tests。

critical:
  Codex planning + human approval + staged execution + maximum verification。
```

## Escalation Rules

升级规则负责处理失败、不确定性和风险变化。

```text
CodeBuddy 第一次低风险失败 -> 可以重试一次。
CodeBuddy 第二次失败 -> 必须升级 Codex。
medium 失败 -> Codex 判断是否重试。
high 默认不自动重试，需要 replanning。
critical 不自动执行失败后的下一步。
Provider 不能降低风险。
Memory 可以调整评分，但不能绕过安全规则。
```

## 与其他模块的关系

Capability Resolver 依赖：

```text
Risk Evaluator
AgentProfile
HAS Skill Registry
Provider Matrix
Project Policy
Context Manager
Memory Center
```

输出：

```text
ExecutionPlan
ExecutionRequest 的核心路由参数
Review Level
Fallback / Escalation Strategy
```

## 核心原则

```text
贵模型负责判断，便宜模型负责执行。
低成本执行必须有边界。
风险越高，Codex 参与越深。
经验可以影响分数，不能放宽安全边界。
```

