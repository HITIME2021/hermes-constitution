# AgentProfile and HAS Skill Registry

## AgentProfile

AgentProfile is a role. It is not an execution provider.

Example:

```text
Frontend Developer = role
CodeBuddy = executor
Codex = planner / reviewer
```

Roles are imported from `msitarzewski/agency-agents` through an adapter.

```text
agency-agents Markdown
  -> Agency Adapter
  -> Hermes AgentProfile
```

## AgentProfile Fields

```yaml
agent_profile:
  id: frontend-developer
  name: Frontend Developer
  division: engineering
  source:
    type: github
    repo: msitarzewski/agency-agents
    path: engineering/frontend-developer.md
  persona:
    enabled: true
    prompt_ref: original_markdown
  responsibilities:
    - ui_implementation
    - component_development
    - accessibility
  capability_tags:
    - react
    - typescript
    - css
  task_affinity:
    implementation: high
    architecture: medium
    review: medium
  risk_policy:
    default_risk: medium
    can_handle_independent_risk: low
    requires_codex_planning_from: medium
    requires_codex_review_from: medium
  provider_policy:
    preferred_executor: codebuddy
    planner: codex
    reviewer: codex
```

## HAS Skill Registry

Skill is a reusable method. It is not a tool and not a permission grant.

```text
Skill = how to do a task class
Tool = what action can be performed
AgentProfile = who is responsible
Provider = who executes
Workflow = when it happens
```

## Skill Types

```text
implementation_skill
review_skill
testing_skill
planning_skill
migration_skill
memory_skill
```

## Skill Fields

```yaml
skill:
  id: react-component-builder
  name: React Component Builder
  version: 0.1.0
  applies_to:
    task_types:
      - frontend_implementation
    roles:
      - frontend-developer
    capability_tags:
      - react
      - typescript
  provider_policy:
    usable_by:
      - codex
      - codebuddy
    preferred_executor: codebuddy
    requires_codex_review_from_risk: medium
  risk_policy:
    min_risk: trivial
    max_independent_risk: low
    forbidden_domains:
      - security
      - payment
      - migration
  context_requirements:
    required:
      - task_goal
      - target_files
      - acceptance_criteria
    preferred_scope: file_set
  outputs:
    must_report:
      - files_changed
      - assumptions
      - risks
      - tests_run
```

## Skill Execution Packet

CodeBuddy receives a scoped packet, not the full registry.

```yaml
skill_execution_packet:
  skill_id: react-component-builder
  role: frontend-developer
  provider: codebuddy
  goal: "Add loading state to SubmitButton."
  allowed_files:
    - src/components/SubmitButton.tsx
  forbidden_actions:
    - "Do not change public props unless required."
    - "Do not add dependencies."
  steps:
    - "Inspect existing component style."
    - "Implement minimal change."
    - "Add or update focused tests."
  acceptance_criteria:
    - "Button shows loading state when isLoading=true."
  report_format:
    - files_changed
    - summary
    - assumptions
    - tests_run
    - risks
```

