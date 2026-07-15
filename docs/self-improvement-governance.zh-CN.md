# Self-Improvement Governance 自我改进治理

Self-Improvement Governance 定义 Hermes 如何从会话中学习，并改进 skill、profile、
prompt 或 policy，同时避免静默改变可信执行行为。

## 核心规则

<!-- snapshot:block id="self-improvement-governance.zh-CN" section="Self-Improvement Governance" priority="62" -->
Self-improvement analysis 是允许的。Self-improvement writes 是
authority-bearing effects。

Hermes 可以根据重复失败、用户纠正、流程摩擦、过期路径、低质量 prompt、缺失 skill
或重复 review finding，自动生成 `SelfImprovementCandidate`。

自动 candidate generation 可以发生在 background review、gateway、DM、Kanban 或 TUI
上下文中，但自动 patch application 受到限制。

```yaml
self_improvement:
  allowed_automatic_actions:
    - analyze_session
    - summarize_repeated_failures
    - propose_skill_update
    - propose_memory_candidate
    - propose_policy_update
    - create_self_improvement_candidate

  writes_require_authority:
    - patch_SKILL_md
    - patch_agent_profile
    - patch_runtime_config
    - patch_command_handler
    - write_memory
    - modify_provider_policy
```

Gateway startup、DM cold start、session startup verification、未认证 channel、
新 pairing channel，以及任何 untrusted entrypoint，都不得直接 apply
self-improvement patch。它们只能创建或报告 candidate。

一个 `SelfImprovementCandidate` 应包含：

```yaml
self_improvement_candidate:
  id: si-YYYYMMDD-HHMMSS
  source_channel: weixin_dm
  trigger: repeated_constitution_path_confusion
  target:
    type: skill
    path: ~/.hermes/skills/example/SKILL.md
  proposal:
    summary: Update stale constitution source path.
    patch_summary: Replace old path with production path.
  evidence:
    - user correction
    - gateway log line
    - failed smoke test
  risk_level: low
  requires_approval: true
  allowed_apply_channels:
    - tui
    - kanban_task
  forbidden_apply_channels:
    - gateway_startup
    - dm_cold_start
```

Apply self-improvement candidate 必须具备：

- trusted execution channel
- 声明过的 target scope
- 可见 diff 或 patch summary
- approval state
- 必要时的 rollback 或 backup plan
- verification 或 smoke test plan
- audit evidence

低风险纯文本 skill patch 可以批处理或通过 Kanban surfaced，但不得从 gateway startup
或 DM cold start 直接 apply。Runtime code patch、provider policy change、memory write
和 command handler change 需要更强 review 和明确 operator approval。
<!-- /snapshot:block -->

## 非目标

本策略不禁用 Hermes 学习。

本策略不阻止可信 TUI 或 Kanban self-improvement workflow。

本策略不允许后台静默修改 skill、runtime code、memory 或 provider policy。
