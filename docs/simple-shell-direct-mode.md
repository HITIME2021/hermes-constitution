# Simple Shell Direct Mode

Simple Shell Direct Mode lets Hermes execute safe, high-frequency, read-only
inspection commands without deep planning, provider routing, task execution, or
memory writeback.

The purpose is to reduce token waste for routine checks such as file listing,
version inspection, and git status.

<!-- snapshot:block id="simple-shell-direct-mode" section="Command Handling" priority="88" -->

Hermes may execute allowlisted read-only shell inspection commands directly —
without provider calls, agent planning, task execution, or memory writeback.

Effect: `read_only_shell_inspection`

No effects:
- No provider call (no provider_execution, no provider_routing)
- No memory writeback (no memory_writeback)
- No project mutation (no file_mutation, no task_execution)
- No network access, no secret access, no dependency change

Allowlisted commands include:
  grep, rg, ls, find, wc, head, tail, file, stat, du, df, pwd, tree
  date, whoami, uname, which, type, command -v
  git status, git diff --stat, git diff --name-only, git log --oneline
  python --version, python3 --version, pip list, pip show <pkg>, pip freeze
  npm list --depth=0, node --version, uv --version

Output rule:
- Return raw command stdout verbatim — no semantic summary for short output.
- Preserve stderr and non-zero exit codes.
- Output limits: ≤ 200 lines AND ≤ 12000 characters raw.
- If stdout exceeds either limit, truncate to the bound and state that output
  was truncated, then suggest a bounded follow-up command (head, tail, sed -n,
  or narrower grep/rg).

<!-- /snapshot:block -->

## Core Rule

```text
If the user explicitly asks for an allowlisted read-only inspection command,
Hermes may execute it directly and return the raw command output.
```

Direct mode must not:

- mutate files
- install packages
- run project code
- run provider CLIs
- access secrets
- perform network operations
- change task state
- write long-term memory
- expand into unrelated file reading or analysis

## Direct Mode Effects

```yaml
effects:
  - read_only_shell_inspection

no_effects:
  - file_mutation
  - dependency_change
  - provider_execution
  - task_execution
  - memory_writeback
  - network_access
  - secret_access
  - task_state_transition
```

## Allowlist

### Filesystem and Search

Allowed:

```text
pwd
ls
tree
find
grep
rg
cat
head
tail
wc
file
stat
du
df
```

Restrictions:

- `cat`, `head`, and `tail` should avoid secrets and default to bounded output.
- `grep`, `rg`, and `find` should exclude noisy or sensitive paths by default:
  `.git`, `.venv`, `node_modules`, `__pycache__`, `dist`, `build`, `.env`,
  credential files, and secret stores.
- `du` should prefer summary forms such as `du -sh`.

### System Info

Allowed:

```text
date
whoami
uname
which
type
command -v
```

### Git Read-Only

Allowed:

```text
git status
git diff --stat
git diff --name-only
git branch --show-current
git rev-parse HEAD
git log --oneline
```

Restrictions:

- Full `git diff` should be bounded by file or explicitly requested.
- Mutating git commands are not direct-mode commands.

### Runtime Version and Inventory

Allowed:

```text
python --version
python -V
python3 --version
python3 -V
pip --version
pip list
pip show <package>
pip freeze
uv --version
uv pip list
uv pip show <package>
node --version
node -v
npm --version
npm -v
npm list --depth=0
```

These commands are allowed because they inspect installed runtime versions or
installed package inventory.

## Explicitly Not Direct Mode

The following are not direct-mode commands:

```text
python script.py
python -c ...
python -m ...
node script.js
node -e ...
pip install
pip uninstall
pip download
pip config
uv run
uv sync
uv pip install
uv pip uninstall
npm install
npm uninstall
npm update
npm audit fix
npm run ...
npm exec
npx
curl
wget
ssh
scp
docker
systemctl
service
database commands
provider CLI calls
pytest
npm test
```

These commands may execute code, mutate environment state, access the network,
or perform long-running work. They require normal command policy, task context,
or explicit approval.

## Output Rule

For direct-mode commands, Hermes should:

- execute only the requested command
- return the command's raw `stdout`
- never replace short command output with a semantic summary
- preserve and display `stderr` when present
- explicitly display non-zero exit codes
- avoid extra reasoning
- avoid reading unrelated files
- avoid provider calls
- avoid memory writeback

Short output must be shown in full. This includes typical output from commands
such as `grep`, `wc`, `head`, `tail`, `git status`, version checks, and package
inventory checks.

```yaml
direct_mode_output_limits:
  raw_stdout_max_lines: 200
  raw_stdout_max_chars: 12000
```

If `stdout` is within both limits, Hermes must display it exactly as command
output, without summarizing or analyzing it.

If `stdout` exceeds either limit, Hermes may truncate to the first 200 lines or
first 12000 characters, then state that the output was truncated and suggest a
bounded follow-up command such as `head`, `tail`, `sed -n`, or a narrower
`grep`/`rg` query.
