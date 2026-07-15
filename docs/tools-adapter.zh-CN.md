# Tools Adapter 工具适配器

Tools Adapter 定义 Hermes 如何包装外部工具，使它们能通过统一的 policy、
evidence 和 artifact intake 接口被调用。

本文档不绑定某个具体工具。具体工具可以作为示例出现，但适配器合约基于 usage
mode 和 effects，而不是工具品牌。

## 核心规则

<!-- snapshot:block id="tools-adapter.zh-CN" section="Tools Adapter" priority="83" -->
Hermes 调用外部工具时，必须通过声明过的 Tools Adapter entry。

```text
Tools Layer 决定一次工具调用属于什么类别。
Tools Adapter 决定工具如何调用、如何限定 scope、如何取证、如何映射输出。
Artifact Intake Gate 决定 Frontend artifact 是否可以影响执行。
Provider Adapter policy 决定 provider 工作是否可以执行。
```

一个 Tools Adapter entry 必须声明：

```yaml
tool_adapter:
  tool_name: example-tool
  execution_plane: WSL
  invocation:
    command: example
    args_template:
      - subcommand
  supported_modes:
    - read_only_inspection
    - artifact_generation
    - backend_effect
  default_classification: backend_tool_until_mode_is_proven_artifact_only
  allowed_read_only_commands:
    - "--help"
    - "version"
    - "check"
  artifact_outputs:
    - spec.md
    - plan.md
    - tasks.md
    - checklist.md
  prohibited_without_execution_request:
    - project_mutation
    - dependency_install
    - provider_routing
    - workflow_execution
    - self_upgrade
    - memory_writeback
```

只读检查命令可以用于验证工具是否可用、版本、能力和帮助文本。其输出只是
environment evidence。工具发现结果不得自动修改 Hermes Provider Policy，不得
批准新 provider，也不得扩大 execution scope。

Artifact generation mode 可以产生规划 artifact。这些 artifact 仍然只是 input
evidence，必须先通过 Artifact Intake Gate，才能影响 live execution。

Backend effect mode 可能创建文件、安装模板、运行 workflow、更新 integration、
改变 runtime state，或产生其他可观察副作用。Backend effect mode 必须具备
`ExecutionRequest`、声明过的 scope、必要 approval state、stop conditions 和
audit-grade evidence。

Hermes 必须对每一次调用重新分类。一个工具在 read-only mode 下安全，并不代表
它在 init、workflow、upgrade、integration 或 project mutation 模式下也安全。

示例 adapter entry：

```yaml
tool_adapter:
  tool_name: spec-kit/specify
  execution_plane: WSL
  invocation:
    command: uvx
    args_prefix:
      - "--from"
      - "git+https://github.com/github/spec-kit.git"
      - "specify"
  verified_capability_evidence:
    cli_version: "0.12.16.dev0"
    python: "3.11.15"
    platform: "Linux x86_64"
  read_only_inspection:
    - "--help"
    - "version"
    - "check"
    - "init --help"
  artifact_mode:
    class: frontend_tool
    authority: input_evidence_only
    requires: Artifact Intake Gate
  init_or_workflow_mode:
    class: backend_tool
    requires: ExecutionRequest
    recommended_scope: isolated_workspace_or_explicit_project_scope
  forbidden_without_explicit_approval:
    - "init --here"
    - "init ."
    - "init --here --force"
    - "self upgrade"
    - "workflow run"
```

工具生成的 integration 文件，例如 Codex 或 CodeBuddy command templates，不会改变
Hermes Provider Policy。Provider 角色仍由 Project Policy、Provider Adapter policy
和 operator approval 管控。
<!-- /snapshot:block -->

## Adapter 职责

Tools Adapter 应当：

- 构造精确命令调用
- 声明 execution plane
- 分类 invocation mode
- 绑定 allowed 和 forbidden scope
- 捕获 stdout、stderr、exit code、生成文件和 diagnostics
- 将 artifact 映射为 Hermes artifact type
- 在可用时记录工具版本和能力证据
- 暴露 stop conditions，避免浪费 provider token 的重试循环

## 非目标

Tools Adapter 不会让工具输出获得权威。

Tools Adapter 不会批准工具发现到的 provider。

Tools Adapter 不替代 Artifact Intake Gate、Provider Adapter policy、Review Gate、
Human Intervention policy 或 Memory Writeback approval。
