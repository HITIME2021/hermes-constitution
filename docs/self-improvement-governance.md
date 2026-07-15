# Self-Improvement Governance

Self-Improvement Governance defines how Hermes may learn from sessions and
improve skills, profiles, prompts, or policies without silently changing trusted
execution behavior.

## Core Rule

<!-- snapshot:block id="self-improvement-governance" section="Self-Improvement Governance" priority="61" -->
Self-improvement analysis is allowed. Self-improvement writes are
authority-bearing effects.

Hermes may automatically generate `SelfImprovementCandidate` records from
repeated failures, user corrections, workflow friction, stale paths, poor
prompts, missing skills, or recurring review findings.

Automatic candidate generation may occur in background review, gateway, DM,
Kanban, or TUI contexts, but automatic patch application is restricted.

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

Gateway startup, DM cold start, session startup verification, unauthenticated
channels, newly paired channels, and any untrusted entrypoint must not directly
apply self-improvement patches. They may only create or report candidates.

A `SelfImprovementCandidate` should include:

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

Applying a self-improvement candidate requires:

- trusted execution channel
- declared target scope
- visible diff or patch summary
- approval state
- rollback or backup plan when relevant
- verification or smoke test plan
- audit evidence

Low-risk text-only skill patches may be batched or surfaced through Kanban, but
they must not be applied from gateway startup or DM cold start. Runtime code
patches, provider policy changes, memory writes, and command handler changes
require stronger review and explicit operator approval.
<!-- /snapshot:block -->

## Non-Goals

This policy does not disable Hermes learning.

This policy does not prevent trusted TUI or Kanban self-improvement workflows.

This policy does not allow silent background mutation of skills, runtime code,
memory, or provider policy.
