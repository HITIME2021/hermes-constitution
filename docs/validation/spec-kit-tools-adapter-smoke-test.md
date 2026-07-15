# Run-SPEC-001: spec-kit Tools Adapter Smoke Test

Date: 2026-07-15

Constitution release: v0.3.1

Constitution version at test time: `8de33db` or later v0.3.1 snapshot including
Workspace Layout Policy and Tools Adapter runtime-effects policy.

Status: PASS

## Purpose

Validate that Hermes can govern `spec-kit/specify` through the generic Tools
Adapter model without treating the tool as execution authority.

The test specifically validated:

- read-only inspection command classification
- backend-effect classification for `specify init`
- Workspace Layout isolation under `~/projects/labs`
- production repository non-mutation
- runtime environment effects recording
- Artifact Intake Gate readiness for generated artifacts
- Provider Policy isolation from `specify check` and integration output

## Workspace

```yaml
workspace:
  root: ~/projects/labs/spec-kit-smoke-001
  class: lab
  purpose: tool_validation
  target_project: none
```

Production repositories were kept under:

```text
~/projects/production/hermes-constitution
~/projects/production/GitHub-Trend-Intelligence-Platform
```

Both production repositories were verified as unmodified after the smoke test.

## Commands

Read-only inspection phase:

```bash
uvx --from git+https://github.com/github/spec-kit.git specify --help
uvx --from git+https://github.com/github/spec-kit.git specify version
uvx --from git+https://github.com/github/spec-kit.git specify check
uvx --from git+https://github.com/github/spec-kit.git specify init --help
```

Bounded init phase:

```bash
uvx --from git+https://github.com/github/spec-kit.git specify init --here --integration codex
```

The bounded init phase required operator approval because it is a backend effect.
Approval was limited to `~/projects/labs/spec-kit-smoke-001`.

## Tool Classification

```yaml
tool_adapter:
  tool_name: spec-kit/specify
  invocation:
    command: uvx
    args_prefix:
      - "--from"
      - "git+https://github.com/github/spec-kit.git"
      - "specify"

read_only_inspection:
  commands:
    - "--help"
    - "version"
    - "check"
    - "init --help"
  result: pass

backend_effect:
  command: "init --here --integration codex"
  result: pass_after_operator_approval
  allowed_scope: ~/projects/labs/spec-kit-smoke-001
```

## Runtime Effects

```yaml
runtime_effects:
  network_access:
    occurred: true
    description: uvx fetched package content from git+https://github.com/github/spec-kit.git
    scope: tool_fetch_only

  uv_cache_write:
    occurred: likely
    description: spec-kit package content was written into the uv runtime cache
    scope: uv_runtime_cache

  filesystem_mutation:
    occurred: true
    scope: ~/projects/labs/spec-kit-smoke-001 only
    files_created: 25
    production_modified: false

  script_execution:
    occurred: true
    description: init set executable bits on generated scripts
    scope: labs directory only

  dependency_change:
    occurred: false
    description: no dependency manifest or lockfile was modified

  credential_access:
    occurred: false

  memory_writeback:
    occurred: false

  git_operations:
    occurred: false

  workflow_execution:
    occurred: false
    description: workflow files were generated or registered but not run
```

## Artifact Intake Result

Generated spec-kit outputs are not execution authority.

They are classified as:

```text
Frontend artifacts -> input evidence only
Backend scaffold effects -> generated project/tool infrastructure inside labs scope
```

Any future use of generated `spec.md`, `plan.md`, `tasks.md`, checklist files,
scripts, or workflow files must pass Artifact Intake Gate before influencing
live execution.

## Provider Policy Result

`specify check` detected available tools including Codex CLI, CodeBuddy, and
Hermes Agent. This output was treated only as environment evidence.

It did not:

- modify Hermes Provider Policy
- approve new providers
- change provider priority
- authorize execution
- bypass Codex review or operator approval

Generated Codex integration files also do not change Hermes provider roles.

## Stop Conditions

```yaml
stop_conditions:
  production_path_modified: clear
  init_without_operator_approval: clear
  force_init_used: clear
  workflow_run_executed: clear
  self_upgrade_executed: clear
  provider_policy_changed_by_tool_discovery: clear
  artifact_intake_bypassed: clear
  memory_writeback: clear
```

## Final Status

```yaml
final_status: pass
summary: |
  Run-SPEC-001 passed.

  spec-kit/specify can be used by Hermes as a governed external tool under
  Tools Adapter. Read-only inspection works. Backend-effect init works only
  after operator approval and only inside the labs workspace.

  The tool is ready for normal Hermes-assisted use as a Frontend Tool source of
  planning artifacts, with Artifact Intake Gate required before any live
  execution.
```

## Operator Summary

Run-SPEC-001 证明 `spec-kit/specify` 已经可以进入正常使用，但它不是权威来源：

- 可以用它生成或维护 spec/plan/tasks 类 artifact。
- 产物必须进入 Artifact Intake Gate。
- `init`、workflow、integration、bundle、extension、self upgrade 等写入或执行行为
  必须按 Backend Tool 管控。
- `uvx` 调用会有 network/cache runtime effects，不能写成完全无副作用。
- 生产仓库不能作为 spec-kit 实验 scratch space；实验应放在 `~/projects/labs`。
