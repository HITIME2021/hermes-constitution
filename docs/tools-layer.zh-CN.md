# Tools Layer 工具层

Tools Layer 定义 Hermes 如何对待外部工具。工具可以帮助规划、生成 artifact、
执行、验证、审查和报告，但工具输出不会自动获得宪法权威。

Hermes 仍然是 orchestrator、policy owner 和最终 execution authority。

## 核心规则

<!-- snapshot:block id="tools-layer-classification.zh-CN" section="Tools Layer" priority="81" -->
Hermes 按工具产生的结果类型分类：

```text
Frontend Tools produce artifacts.
Frontend Tool output is input evidence, not authority.

Backend Tools produce effects.
Backend Tool execution is authority-bearing effect and requires scope control.

Hermes governs both.
```

Frontend Tools 是 artifact-producing tools。它们可以帮助生成 specification、
plan、task list、checklist、design brief、risk note、clarifying questions
或其他面向操作者的规划 artifact。它们的输出只是 advisory input，不是已批准
的 `ExecutionRequest`，不是 provider routing authority，不是 review verdict，
也不是 memory writeback approval。

Backend Tools 是 effect-producing tools。它们可能读取项目状态、执行命令、修改
文件、运行测试、调用 provider CLI/API、改变 runtime state、部署、迁移数据，或
产生其他可观察副作用。Backend Tools 必须只能在 Hermes `ExecutionRequest` 下
运行，并声明 `allowed_scope`、`forbidden_scope`、`execution_mode`、approval
state、stop conditions 和 evidence requirements。

Ambiguous Tools 默认按 Backend Tool 处理，直到 Hermes 明确把当前 usage mode
分类为 artifact-only。如果同一个工具既能生成 artifact，又能执行 shell command
或 mutation，Hermes 必须拆分 usage mode：

```text
artifact mode  -> Frontend Tool rules
execution mode -> Backend Tool rules
```

Frontend Tool 输出进入执行前必须经过 Hermes intake：

```text
validate -> normalize -> map -> approval gate
```

Hermes 不得允许 Frontend Tool 隐式批准任务、决定 provider routing、绕过 review、
绕过 stop conditions、写入长期 memory、修改目标项目文件，或通过 workflow/shell
间接产生执行效果。

工具分类基于当前 usage mode，而不是工具品牌。某个工具只用于生成 planning
artifacts 时，按 Frontend Tool 处理。同一个工具如果用于运行命令、修改文件、调用
provider 或改变状态，则该次调用按 Backend Tool 处理。
<!-- /snapshot:block -->

## Frontend Tools

Frontend Tools 帮助 Hermes 和操作者理解要构建或审查什么。

示例：

- specification generators
- design exporters
- document analyzers
- prompt distillers
- checklist generators
- clarification assistants
- project scaffolding tools when used in dry-run or artifact-only mode

预期输出：

```text
spec.md
plan.md
tasks.md
checklist.md
design_brief.md
risk_analysis.md
clarifying_questions.md
```

Frontend artifacts 可以成为 evidence、context，或 Hermes 对象的 draft input，但
在 Hermes validate 和 normalize 之前，不会成为 execution authority。

## Backend Tools

Backend Tools 执行会产生可观察效果的工作。

示例：

- Codex CLI 在 task contract 下做 planning、review 或 repair
- CodeBuddy CLI 修改代码
- shell commands
- `pytest`
- `git`
- `uv`
- `npm`
- database migration tools
- crawler runners
- deployment tools

Backend Tool 使用必须能在 run evidence 中追溯。当 Backend Tool 需要扩大 scope、
新增依赖、读取 credential、修改 provider 配置、执行 destructive operation、
执行 network-sensitive operation，或需要 interactive confirmation 时，Hermes
必须遵守既有 approval 和 stop-condition 规则。

## 工具示例

工具示例只是说明，不是特殊规则。Hermes 必须按每次调用的 usage mode 分类：

```text
artifact-only specification generation -> Frontend Tool
workflow shell execution               -> Backend Tool
dry-run design export                  -> Frontend Tool
project file generation                -> Backend Tool
read-only inspection command           -> Backend Tool, possibly Simple Shell Direct Mode
```

例如，一个 specification toolkit 在只输出 `spec.md`、`plan.md`、`tasks.md`、
`checklist.md` 或 clarifying questions 时，可以是 Frontend Tool。如果同一个
toolkit 执行 workflow shell steps、创建项目文件、安装依赖，或改变 runtime state，
该次调用就是 Backend Tool 行为。

## Tool Intake Rules

每次工具调用前，Hermes 应声明工具分类：

```yaml
tool_classification:
  tool_name: example-tool
  mode: artifact_generation_only
  class: frontend_tool
  expected_outputs:
    - spec.md
    - plan.md
    - tasks.md
  prohibited_effects:
    - live_execution
    - provider_routing
    - approval_decision
    - shell_mutation
    - memory_writeback
```

如果 usage mode 改变，分类必须重新评估。一次 artifact-only 分类不授权之后的
execution 行为。

## 非目标

Tools Layer 不替代 Provider Adapter policy。会执行 provider 工作的工具仍然需要
provider routing、adapter enforcement、review 和 evidence。

Tools Layer 不让 Frontend Tool artifact 获得权威。Live execution 前，Hermes 仍
必须执行 artifact intake 和 operator approval。
