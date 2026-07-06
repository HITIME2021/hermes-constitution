# Provider Auth Policy

Provider authentication is an operator-controlled trust boundary. Hermes may
verify provider readiness, but it must not manage, inspect, copy, store, or
synthesize provider credentials.

## Core Rule

```text
Operator owns login.
Hermes verifies readiness.
Adapters invoke providers through existing authorized CLI sessions.
Secrets never enter provider context or memory.
```

## Codex Authentication

For this operator environment, Codex CLI should use the user's ChatGPT/OAuth
login session in WSL.

```yaml
codex_auth_policy:
  preferred_auth_mode: chatgpt_oauth
  billing_intent: chatgpt_plus_entitlement
  cli_login_command: codex login
  api_key_default: forbidden
  execution_plane: wsl
```

OpenAI API keys and ChatGPT/Plus OAuth are separate authentication and billing
paths. Hermes must not assume that an API key is required for Codex CLI when the
operator intends to use ChatGPT/Plus entitlement.

Allowed readiness checks:

```text
which codex
codex --version
codex doctor
codex login --help
minimal codex exec smoke test, only when explicitly approved
```

Forbidden by default:

```text
read Codex token files
read OpenAI API keys
read .env for provider credentials
inject OPENAI_API_KEY into Codex execution
copy credentials between Windows, WSL, Desktop, or CLI surfaces
store auth tokens in memory
pass auth material to CodeBuddy or other providers
run codex logout unless explicitly requested
run codex update as part of auth readiness
auto-confirm Codex trust/auth/permission prompts
```

Hermes may invoke Codex CLI through the existing WSL OAuth session. If
`OPENAI_API_KEY`, `OPENAI_BASE_URL`, `OPENAI_ORG`, or `OPENAI_PROJECT` is present
in the environment, Hermes must treat it as a possible API-billing path and ask
the operator before using it for Codex.

## CodeBuddy Authentication

CodeBuddy authentication is also operator-owned.

Hermes may check:

```text
which codebuddy
codebuddy --version
codebuddy auth status, if supported
minimal CodeBuddy smoke test, only when explicitly approved
```

Hermes must not read CodeBuddy credential files, tokens, session stores, or
configuration secrets.

Forbidden by default:

```text
read ~/.codebuddy/*
read ~/.codebuddy/settings.json
read ~/.codebuddy/user-state.json
read CodeBuddy auth files
read CodeBuddy token files
read CodeBuddy session stores
read CodeBuddy config secrets
infer auth mode from credential file contents
copy CodeBuddy credentials to another provider
store CodeBuddy auth material in memory
auto-confirm CodeBuddy trust/auth/permission prompts
```

Hermes may verify CodeBuddy readiness through CLI behavior, not credential file
inspection. If `codebuddy auth status` is unavailable or hangs, use an explicitly
approved minimal smoke test instead of reading files under `~/.codebuddy/`.

## Auth Readiness Record

Hermes may record non-secret readiness metadata:

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

Do not record:

- API keys
- OAuth tokens
- refresh tokens
- cookies
- credential file contents
- `.env` values
- screenshots of login pages

## Surface Separation

ChatGPT Desktop, Codex CLI, Codex web/cloud, and API keys are separate surfaces.
They may use the same user account, but they maintain separate login/session
state and may have separate billing behavior.

Hermes must not assume that one surface's authentication is available to another
surface unless verified in that execution plane.

## Hard Rules

- Auth setup is a bootstrap/operator responsibility.
- Hermes must not automatically log in, log out, rotate, copy, or inspect credentials.
- Hermes must not default Codex CLI to API-key billing when the operator intends ChatGPT/Plus OAuth.
- Hermes must not read provider home credential/config directories such as `~/.codebuddy/`.
- Hermes must not auto-confirm provider trust, auth, permission, install, or elevated-action prompts.
- Provider auth failures move tasks to `blocked`, not `failed`.
- Auth checks must be read-only unless the user explicitly approves a smoke test.
- Auth material must never enter Memory Center.
