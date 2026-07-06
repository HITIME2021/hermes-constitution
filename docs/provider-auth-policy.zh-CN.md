# Provider 认证策略

Provider 认证是由操作者控制的信任边界。Hermes 可以验证 Provider 是否就绪，但不得管理、查看、复制、保存或合成 Provider 凭据。

## 核心规则

```text
操作者负责登录。
Hermes 只验证就绪状态。
Adapter 通过已有授权 CLI 会话调用 Provider。
Secrets 永不进入 provider context 或 memory。
```

## Codex 认证

在这个操作者环境中，Codex CLI 应使用 WSL 中用户自己的 ChatGPT/OAuth 登录会话。

```yaml
codex_auth_policy:
  preferred_auth_mode: chatgpt_oauth
  billing_intent: chatgpt_plus_entitlement
  cli_login_command: codex login
  api_key_default: forbidden
  execution_plane: wsl
```

OpenAI API key 和 ChatGPT/Plus OAuth 是两条不同的认证与计费路径。当操作者希望使用 ChatGPT/Plus 权益时，Hermes 不得假设 Codex CLI 必须使用 API key。

允许的就绪检查：

```text
which codex
codex --version
codex doctor
codex login --help
最小 codex exec smoke test，仅在用户明确批准时执行
```

默认禁止：

```text
读取 Codex token 文件
读取 OpenAI API key
读取 .env 中的 Provider 凭据
向 Codex 执行注入 OPENAI_API_KEY
在 Windows、WSL、Desktop、CLI 之间复制凭据
把 auth token 写入 memory
把认证材料传给 CodeBuddy 或其他 Provider
执行 codex logout，除非用户明确要求
把 codex update 当成 auth readiness 的一部分
自动确认 Codex trust/auth/permission prompt
```

Hermes 可以通过 WSL 中已有的 OAuth 会话调用 Codex CLI。如果环境里存在 `OPENAI_API_KEY`、`OPENAI_BASE_URL`、`OPENAI_ORG` 或 `OPENAI_PROJECT`，Hermes 必须把它视为可能走 API 计费路径，并在用于 Codex 前询问操作者。

## CodeBuddy 认证

CodeBuddy 认证同样由操作者负责。

Hermes 可以检查：

```text
which codebuddy
codebuddy --version
codebuddy auth status，如果支持
最小 CodeBuddy smoke test，仅在用户明确批准时执行
```

Hermes 不得读取 CodeBuddy 凭据文件、token、session store 或配置 secrets。

默认禁止：

```text
读取 ~/.codebuddy/*
读取 ~/.codebuddy/settings.json
读取 ~/.codebuddy/user-state.json
读取 CodeBuddy auth 文件
读取 CodeBuddy token 文件
读取 CodeBuddy session stores
读取 CodeBuddy config secrets
根据 credential 文件内容推断 auth mode
把 CodeBuddy 凭据复制给其他 Provider
把 CodeBuddy auth material 写入 memory
自动确认 CodeBuddy trust/auth/permission prompt
```

Hermes 应通过 CLI 行为验证 CodeBuddy readiness，而不是检查凭据文件。如果 `codebuddy auth status` 不可用或卡住，应使用用户明确批准的最小 smoke test，而不是读取 `~/.codebuddy/` 下的文件。

## Auth Readiness Record

Hermes 可以记录非敏感就绪元数据：

```yaml
provider_auth_readiness:
  codex:
    path: /home/hitime/.local/bin/codex
    version: codex-cli 0.142.5
    auth_mode: chatgpt_oauth
    auth_status: verified_by_user_login
    credential_material_recorded: false
  codebuddy:
    path: /home/hitime/.local/bin/codebuddy
    version: 2.115.0
    auth_status: verified_by_user_or_smoke_test
    credential_material_recorded: false
```

不得记录：

- API keys
- OAuth tokens
- refresh tokens
- cookies
- credential 文件内容
- `.env` 值
- 登录页面截图

## Surface 分离

ChatGPT Desktop、Codex CLI、Codex web/cloud 和 API key 是不同 surface。它们可以使用同一个用户账号，但各自维护登录/session 状态，并且可能具有不同计费行为。

除非在对应 execution plane 中验证过，否则 Hermes 不得假设某个 surface 的认证状态可以被另一个 surface 使用。

## 硬规则

- Auth setup 是 bootstrap/operator 责任。
- Hermes 不得自动登录、登出、轮换、复制或查看凭据。
- 当操作者希望使用 ChatGPT/Plus OAuth 时，Hermes 不得默认让 Codex CLI 走 API-key 计费。
- Hermes 不得读取 `~/.codebuddy/` 这类 Provider home credential/config 目录。
- Hermes 不得自动确认 Provider trust、auth、permission、install 或 elevated-action prompt。
- Provider auth failure 应将任务转为 `blocked`，不是 `failed`。
- Auth check 默认只读；smoke test 必须由用户明确批准。
- Auth material 永不进入 Memory Center。
