# Project Policy

Project Policy is the project constitution. It defines what Hermes may do in a
specific project.

Project Policy has priority over AgentProfile, Skill, and Provider preference.
When rules conflict, the stricter rule wins.

## v0.1 Decisions

- CodeBuddy is forbidden by default from modifying `auth`, `security`, `db`, and
  `infra` domains.
- New dependencies require approval by default.
- Protected domain changes are at least high risk.
- Dependency changes are at least medium risk.
- Secrets, credentials, and tokens are critical risk.

## Policy Areas

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

File scopes should be classified as:

```text
allowed
protected
approval_required
forbidden
```

Protected does not mean unreadable. Codex may need protected context for
planning. CodeBuddy is not allowed to modify protected domains by default.

## Protected Domains

```text
auth
security
database
infrastructure
deployment
ci/cd
```

## Command Policy

Allowed commands are usually verification commands:

```text
npm test
npm run test
npm run typecheck
npm run lint
npm run build
```

Simple read-only inspection commands may use direct mode when they match the
allowlist in [Simple Shell Direct Mode](simple-shell-direct-mode.md). This
includes routine checks such as `pwd`, `ls`, `rg`, `git status`, runtime version
queries, and installed package inventory commands.

Direct mode is not approval for code execution, dependency changes, network
access, provider execution, secret access, or file mutation.

Approval-required commands include:

```text
npm install
pnpm add
yarn add
database migration commands
deployment commands
```

Forbidden command classes include:

```text
destructive deletion
git reset --hard
force push
secret scanning bypass
```

## Dependency Approval Packet

<!-- snapshot:block id="dependency-policy" section="Project Policy" priority="40" -->
Dependency changes are detected by the target project's dependency profile, not
by one hardcoded filename.

Dependency manifests may include, depending on the ecosystem:

```text
requirements.txt
requirements-dev.txt
pyproject.toml
Pipfile
package.json
Cargo.toml
go.mod
```

Lockfiles may include, depending on the ecosystem:

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

Any change to a dependency manifest or lockfile is a dependency change unless
Project Policy explicitly classifies that file differently.
<!-- /snapshot:block -->

When Hermes requests a new dependency, it must provide:

```yaml
dependency_approval_request:
  dependency: example-package
  requested_by: codex
  reason: "Why this dependency is needed."
  alternatives_considered:
    - "Use existing helper."
    - "Manual implementation."
  risk:
    security: low
    maintenance: medium
    runtime_or_bundle_impact: low
  affected_files:
    - package.json
    - lockfile
  recommendation: reject
```

## Hard Rules

- CodeBuddy cannot modify protected domains unless Codex creates a scoped
  execution request and the required approvals are present.
- CodeBuddy cannot add dependencies.
- New dependencies require approval.
- Dependency manifest or lockfile changes require approval by default.
- Project Policy changes are high risk and require Codex review.
- Secrets never enter provider context or memory.
