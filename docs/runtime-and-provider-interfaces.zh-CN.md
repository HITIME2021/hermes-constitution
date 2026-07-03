# 运行时与 Provider 接口

Hermes Core 应该默认 headless。

GUI 很有用，但不应该成为自动化系统的发动机。Hermes 的核心运行时应该可以在 WSL、服务器、CI 或未来的 Web/Desktop 控制台后面运行。

## 运行时策略

```yaml
runtime_policy:
  core_mode: headless
  primary_interface: cli
  secondary_interface: api
  gui_role: optional_control_plane
```

## 为什么要 Headless

Headless core 让 Hermes 更容易：

```text
自动化
测试
重试
审计
在 WSL 运行
在 CI 运行
接 Provider Adapter
失败后恢复
未来接 GUI / Web / Chat / IDE
```

如果 Hermes 的核心流程依赖 GUI 自动化，它会变得难调试、难并发、难部署、难恢复。

## 接口分工

```text
Hermes Core
  headless runtime，负责 workflow、policy、state、memory、routing。

CLI
  主要自动化接口。

API
  未来服务化和外部集成接口。

GUI / Desktop
  可选的人类驾驶舱。
```

## GUI 的角色

桌面软件或 GUI 仍然有价值，但它应该是控制台，不是底层执行接口。

适合 GUI 的事情：

```text
查看任务状态
查看 diff
批准 high / critical 操作
调试失败任务
检查 memory candidates
与 Codex 做深度协作
```

不应该依赖 GUI 的事情：

```text
任务规划
Provider 路由
CodeBuddy 执行
Codex Review
状态流转
Memory Writeback
```

## Provider 接入策略

Provider 应通过 Adapter 接入：

```text
Hermes Workflow Engine
  -> Capability Resolver
  -> Provider Adapter
  -> Provider CLI/API
  -> Structured Result
  -> Review Gate
```

Hermes Core 不应该直接绑定某个 Provider 的 GUI。

Provider Adapter 还必须负责通道可靠性和错误分类。CLI/API 超时必须被报告为 `transport_error`，不能被当作任务语义失败。

```text
CLI timeout
  -> Provider Adapter
  -> transport_error
  -> bounded retry 或 blocked
  -> 默认不触发 task replanning
```

## Codex Provider

Codex 是高级大脑。

```yaml
codex_provider:
  role: brain_and_senior_engineer
  primary_interface: cli
  secondary_interface: api
  gui_interface: optional_human_collaboration
  default_functions:
    - planning
    - architecture_design
    - algorithm_design
    - risk_evaluation
    - review
    - replanning
    - memory_synthesis
```

Codex 桌面软件适合你做人类协作、架构讨论、复杂 review 和临时探索。Hermes 自动化调用 Codex 时，优先走 CLI/API。

## CodeBuddy Provider

CodeBuddy 是低成本执行者。

```yaml
codebuddy_provider:
  role: low_cost_scoped_executor
  primary_interface: cli
  secondary_interface: api
  gui_interface: optional_debugging
  default_functions:
    - scoped_code_editing
    - code_generation
    - repetitive_refactor
    - simple_bugfix
    - test_writing
```

CodeBuddy 应该接收 Hermes 生成的 scoped execution packet，并返回结构化 execution result。正常自动化不应该依赖 GUI。

## 推荐 CLI 形态

未来 Hermes CLI 可以长这样：

```bash
hermes plan "Add loading state to SubmitButton"
hermes route task-001
hermes execute task-001 --provider codebuddy
hermes review task-001
hermes memory writeback task-001
```

Provider Adapter 内部再负责调用：

```text
codex CLI/API
codebuddy CLI/API
```

Provider 的具体命令格式不应该泄漏到 Hermes Core。

## 边界规则

```text
Hermes controls state.
Provider performs work.
Review Gate decides quality.
GUI helps humans supervise.
```

也就是：

```text
Hermes 控状态。
Provider 干活。
Review Gate 判质量。
GUI 帮人类观察和批准。
```

Provider 通道失败由 Adapter 和 ExecutionAttempt 策略处理。它不能伪装成架构失败、流程失败或任务计划失败。

