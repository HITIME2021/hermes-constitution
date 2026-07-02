# Capability Resolver

Capability Resolver is the routing brain of Hermes. It decides what role,
skills, provider, context scope, and review level are needed for a task.

It should not output only one provider. It should output an execution
composition.

```yaml
planner: codex
executor: codebuddy
reviewer: codex
verifier: local-tools
fallback: codex
```

## Three-Layer Design

```text
1. Hard Filters
2. Scoring
3. Escalation Rules
```

## Hard Filters

Hard filters are non-negotiable.

Examples:

- provider cannot write files -> cannot execute code modification
- provider context limit is too small -> cannot handle full project reasoning
- task touches forbidden project paths -> cannot route to CodeBuddy
- high or critical risk -> cannot be independently handled by CodeBuddy
- secrets, credentials, or destructive actions -> require stronger controls

## Scoring Dimensions

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

## Stable Mode Weights

```yaml
stable:
  role_match: 0.15
  capability_match: 0.25
  skill_match: 0.15
  context_fit: 0.15
  reliability: 0.20
  speed: 0.05
  cost: -0.10
  review_coverage: 0.15
```

## Quality Mode Weights

```yaml
quality:
  role_match: 0.10
  capability_match: 0.20
  skill_match: 0.10
  context_fit: 0.20
  reliability: 0.30
  speed: 0.00
  cost: -0.05
  review_coverage: 0.25
```

## Risk Levels

```text
trivial
low
medium
high
critical
```

## Routing by Risk

```text
trivial:
  executor: CodeBuddy
  review: boundary or none

low:
  executor: CodeBuddy
  review: boundary + automated verification

medium:
  planner: Codex
  executor: CodeBuddy
  reviewer: Codex

high:
  planner: Codex
  executor: CodeBuddy scoped subtasks or Codex
  reviewer: Codex
  approval: optional or required

critical:
  planner: Codex
  executor: step-by-step
  reviewer: Codex
  approval: required
```

## Escalation Rules

- CodeBuddy first failure on low-risk work may be retried once.
- CodeBuddy second failure must escalate to Codex.
- Medium risk failure requires Codex diagnosis before retry.
- High risk work does not auto-retry without replanning.
- Critical work does not auto-execute after failure.
- Provider cannot lower risk level.
- Memory can adjust scoring, but cannot bypass safety rules.

