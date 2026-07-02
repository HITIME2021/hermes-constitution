# Hermes v0.1 Architecture

## Definition

Hermes is a multi-agent automation platform. It receives a user goal, turns it
into structured tasks, selects roles and skills, routes execution to providers,
reviews results, and writes reusable experience into memory.

Hermes is not a single agent. It is an operating layer for agents and execution
providers.

## v0.1 Scope

Included:

- Workflow Engine
- Task Planner
- Risk Evaluator
- Agency Registry
- Capability Resolver
- HAS Skill Registry
- Context Manager
- Memory Center
- Project Policy
- Review Gate
- Execution Protocol
- Codex Provider
- CodeBuddy Provider

Excluded from v0.1:

- Claude Code provider
- production deployment automation
- organization-wide memory
- fully autonomous critical operations

## Layered Architecture

```text
Hermes
|
+-- Workflow Layer
|   +-- Workflow Engine
|   +-- State Manager
|   +-- Static Templates
|   +-- Dynamic Replanner
|
+-- Intelligence Layer
|   +-- Codex Brain
|   +-- Task Planner
|   +-- Risk Evaluator
|   +-- Reviewer
|
+-- Agency Layer
|   +-- Agency Registry
|   +-- agency-agents Adapter
|   +-- AgentProfile
|
+-- Capability Layer
|   +-- Capability Resolver
|   +-- HAS Skill Registry
|   +-- Provider Matrix
|   +-- Routing Policy
|
+-- Context & Memory Layer
|   +-- Context Manager
|   +-- Memory Center
|   +-- Project Knowledge Base
|
+-- Execution Layer
|   +-- Codex Provider
|   +-- CodeBuddy Provider
|
+-- Review Layer
    +-- Review Gate
    +-- Verifier
    +-- Quality Policy
```

## Provider Roles

### Codex / GPT-5.5

Codex is the senior brain.

Responsibilities:

- architecture design
- algorithm design
- task decomposition
- risk evaluation
- provider routing
- code review
- failure analysis
- replanning
- memory synthesis

Avoid using Codex for:

- massive mechanical edits
- repetitive boilerplate
- low-value bulk generation

### CodeBuddy

CodeBuddy is the low-cost executor.

Responsibilities:

- scoped code editing
- code generation
- repetitive refactor
- simple bugfix
- test writing
- file-level changes

CodeBuddy must not independently handle:

- architecture decisions
- ambiguous requirements
- security-sensitive logic
- irreversible operations
- production changes
- cross-system design

## Default Operating Modes

### stable

Default mode.

```text
Codex Plan
  -> CodeBuddy Execute
  -> Local Verify
  -> Codex Review when required
  -> Memory Writeback
```

### quality

Used for important, risky, or core system work.

```text
Codex Deep Plan
  -> Codex Design Review
  -> CodeBuddy Scoped Execution
  -> Local Verify
  -> Codex Code Review
  -> Codex Risk Review
  -> Memory Writeback
```

## External Inputs

### agency-agents

`msitarzewski/agency-agents` is treated as an external role catalog. Hermes does
not directly dispatch raw Markdown agents. It imports and normalizes them into
Hermes `AgentProfile` objects.

### gstack

`garrytan/gstack` is treated as workflow and skill inspiration, especially for
review routing, quality gates, and the software-factory loop:

```text
Think -> Plan -> Build -> Review -> Test -> Ship -> Reflect
```

