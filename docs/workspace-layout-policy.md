# Workspace Layout Policy

Workspace Layout Policy defines where Hermes should place production
repositories, tool experiments, smoke projects, worktrees, and archived test
artifacts in the operator's WSL environment.

## Core Rule

<!-- snapshot:block id="workspace-layout-policy" section="Workspace Layout" priority="57" -->
Hermes must keep production repositories, lab projects, provider worktrees, and
archived experiments in separate workspace subdirectories.

Recommended WSL layout:

```text
~/projects/
  production/
    GitHub-Trend-Intelligence-Platform/
    hermes-constitution/

  labs/
    spec-kit-smoke-001/
    hermes-kanban-provider-smoke/
    hermes-kanban-comment-contract-smoke/

  worktrees/
    GitHub-Trend-Intelligence-Platform-hermes-run-011/
    ...

  archive/
    ...
```

Directory meaning:

```text
~/projects/production/  -> real long-lived repositories
~/projects/labs/        -> smoke tests, tool validation, disposable experiments
~/projects/worktrees/   -> git worktrees and Hermes run workspaces
~/projects/archive/     -> obsolete experiments retained for inspection
```

Hermes must not create smoke, lab, or tool-validation projects directly under
`~/projects`. New experimental projects should go under `~/projects/labs`.
Provider orchestration worktrees should go under `~/projects/worktrees` unless
the operator explicitly chooses another path.

Production repositories must not be used as scratch space for frontend tool
experiments such as spec generation, scaffold trials, or workflow validation.
Tools that may write files, such as project initializers or template
installers, should first be tested in `~/projects/labs/<tool-smoke-name>` or an
explicitly declared isolated workspace.

Hermes must record the workspace class in run evidence:

```yaml
workspace:
  root: ~/projects/labs/spec-kit-smoke-001
  class: lab
  purpose: tool_validation
  target_project: none
```

If a task targets a production repository, Hermes must state that explicitly
and must not infer production status merely from a directory name.
<!-- /snapshot:block -->

## Non-Goals

This policy does not require moving existing repositories immediately.

This policy does not override a task-specific operator path approval.

This policy does not replace `allowed_scope`, `forbidden_scope`, worktree
evidence, or stop-condition checks.
