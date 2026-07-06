# Project Policy 中文说明

Project Policy 是 Hermes 在某个项目里的“项目宪法”。它定义当前项目允许什么、禁止什么、什么需要审批、默认如何验证、什么算高风险。

Project Policy 的优先级高于 AgentProfile、Skill 和 Provider 偏好。

当规则冲突时，采用更严格的规则。

## v0.1 已确认决策

```text
CodeBuddy 默认禁止修改 auth / security / db / infra。
新增依赖默认需要 approval。
protected domain 的修改至少 high 风险。
依赖变更至少 medium 风险。
secrets / credentials / tokens 默认 critical 风险。
```

## Project Policy 管什么

v0.1 先管理 8 类规则：

```text
File Policy
Command Policy
Dependency Policy
Risk Policy
Review Policy
Test Policy
Provider Policy
Memory Policy
```

## File Policy

文件范围分为：

```text
allowed
  可以修改。

protected
  可以被 Codex 读取和分析，但修改需要更高审查。

approval_required
  修改前需要 approval。

forbidden
  不允许读取或不允许写入。
```

protected 不等于 forbidden。比如 `src/auth/**` 可以给 Codex 用于理解架构，但 CodeBuddy 默认不能修改。

## Protected Domains

默认保护域：

```text
auth
security
database
infrastructure
deployment
ci/cd
```

CodeBuddy 默认不能修改这些区域。

如果未来确实需要 CodeBuddy 参与 protected domain，必须满足：

```text
Codex 先规划
任务被拆成极小 scoped subtask
生成明确 ExecutionRequest
通过必要 approval
最终由 Codex Review
```

## Command Policy

通常允许的命令：

```text
npm test
npm run test
npm run typecheck
npm run lint
npm run build
```

需要 approval 的命令：

```text
npm install
pnpm add
yarn add
database migration commands
deployment commands
```

禁止的命令类型：

```text
破坏性删除
git reset --hard
force push
绕过 secret scanning
```

## Dependency Policy

新增依赖默认需要 approval。

<!-- snapshot:block id="dependency-policy.zh-CN" section="Project Policy" priority="41" -->
依赖变更由目标项目的 dependency profile 判断，不由单一硬编码文件名判断。

Dependency manifest 可以包括：

```text
requirements.txt
requirements-dev.txt
pyproject.toml
Pipfile
package.json
Cargo.toml
go.mod
```

Lockfile 可以包括：

```text
uv.lock
poetry.lock
Pipfile.lock
package-lock.json
pnpm-lock.yaml
yarn.lock
Cargo.lock
go.sum
```

除非 Project Policy 明确另行分类，否则任何 dependency manifest 或 lockfile 变更都属于 dependency change。
<!-- /snapshot:block -->

Hermes 如果请求新增依赖，不能只问“要不要加”，而应提供 approval packet：

```text
依赖名称
为什么需要
考虑过哪些替代方案
安全风险
维护风险
运行时或包体积影响
会修改哪些文件
推荐 approve 还是 reject
```

默认拒绝以下依赖请求：

```text
依赖无人维护
存在已知安全问题
与项目已有能力重复
只是为了完成 trivial 任务
```

## Provider Policy

CodeBuddy 是低成本执行者，不是架构决策者。

硬规则：

```text
CodeBuddy max_independent_risk = low。
medium 起必须 Codex planning 或 Codex review。
high / critical 不能由 CodeBuddy 独立执行。
CodeBuddy 不能新增依赖。
CodeBuddy 不能自己扩大 allowed_scope。
CodeBuddy 不能降低 risk_level。
Dependency manifest 或 lockfile 变更默认需要 approval。
```

Codex 默认负责：

```text
medium+ planning
medium+ review
high-risk task handling
memory writeback summary
replanning
```

## Policy 合并顺序

建议顺序：

```text
Global Hermes Policy
  -> Project Policy
  -> Task Policy
  -> User Approval
```

更具体的规则可以收紧约束，但不能绕过上层安全边界。

## 硬规则

```text
Project Policy 优先于 Skill、AgentProfile 和 Provider 偏好。
CodeBuddy 不接触 forbidden 内容。
CodeBuddy 默认不修改 protected domains。
新增依赖默认需要 approval。
auth / security / db / infra 默认 high 风险。
secrets / credentials / tokens 默认 critical 风险。
所有行为变更至少需要一种验证方式。
Project Policy 变更本身是 high 风险，需要 Codex Review。
```
