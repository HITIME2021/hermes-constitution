# 简单 Shell 直通模式

简单 Shell 直通模式允许 Hermes 对安全、高频、只读的检查命令直接执行，不进入深度规划、Provider 路由、任务执行或长期记忆写入。

目标是减少日常检查中的 token 浪费，例如列目录、查版本、看 git 状态。

## 核心规则

```text
当用户明确要求执行白名单内的只读检查命令时，Hermes 可以直接执行，并返回简洁结果。
```

直通模式不得：

- 修改文件
- 安装依赖
- 运行项目代码
- 调用 Provider CLI
- 访问 secrets
- 执行联网操作
- 改变 task state
- 写入长期记忆
- 扩展读取无关文件或做额外分析

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

## 白名单

### 文件系统与搜索

允许：

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

限制：

- `cat`、`head`、`tail` 应避开 secrets，并默认限制输出规模。
- `grep`、`rg`、`find` 默认应排除噪声或敏感路径：`.git`、`.venv`、`node_modules`、`__pycache__`、`dist`、`build`、`.env`、credential 文件和 secret stores。
- `du` 优先使用 `du -sh` 这类 summary 形式。

### 系统信息

允许：

```text
date
whoami
uname
which
type
command -v
```

### Git 只读命令

允许：

```text
git status
git diff --stat
git diff --name-only
git branch --show-current
git rev-parse HEAD
git log --oneline
```

限制：

- 完整 `git diff` 应限制文件范围，或由用户明确要求。
- 会修改 git 状态的命令不属于直通模式。

### Runtime 版本与库清单

允许：

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

这些命令只用于查看已安装 runtime 版本或现有库清单，因此可以直通。

## 明确不属于直通模式

以下命令不属于直通模式：

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

这些命令可能执行代码、修改环境、联网或运行较长任务，必须走普通 Command Policy、任务上下文或显式 approval。

## 输出规则

对于直通命令，Hermes 应：

- 只执行用户请求的命令
- 返回原始输出或简短摘要
- 避免额外推理
- 避免读取无关文件
- 避免调用 Provider
- 避免写入长期记忆

如果输出很大，Hermes 应摘要并询问是否需要有界分段查看。

