# Local Hermes Runtime Patches

This document records local Hermes Agent runtime patches used by this operator.
They are implementation references, not normative constitution policy and not
upstream Hermes Agent source code.

The purpose is to make the local customization auditable and reusable without
pretending that every operator should copy the files verbatim.

## Status

```yaml
patch_scope: local Hermes runtime customization
target_runtime: ~/.hermes/hermes-agent
upstream_policy: do_not_commit_or_push_to_hermes_agent_upstream
restore_manifest: ~/.hermes/local-patches/MANIFEST.md
review_status: local Codex adversarial review completed
risk_after_review: low
safe_to_reference: true
safe_to_blind_copy: false
```

## Adoption Rules

Anyone adapting these patches should:

1. Compare against their local Hermes Agent version.
2. Apply only the patch group they need.
3. Prefer a fresh worktree or backup before copying files.
4. Run the relevant tests.
5. Run Codex or equivalent adversarial review after local adaptation.
6. Avoid overwriting upstream changes blindly.
7. Treat this document as a reference, not as a direct install script.

If an upstream Hermes Agent update touches the same files, the operator must
review the upstream diff before restoring local patch files.

## Patch Groups

### PATCH-GROUP-001: Gateway Entry Guard

Purpose:

- Ensure gateway, DM, mobile, and webhook entrypoints verify the loaded
  constitution snapshot and index before provider execution.
- Fail closed when critical snapshot/index information is unavailable.
- Add cache invalidation when the snapshot index changes.

Local files:

```text
gateway/run.py
tests/test_constitution_commands_extended.py
```

Related constitution records:

- `docs/gateway-entry-guard.md`
- `docs/validation/gateway-dm-session-history-smoke-test.md`

Review status:

```yaml
codex_review: approved_with_notes
risk_after_review: low
known_follow_up:
  - add real gateway integration tests beyond simulator tests
```

### PATCH-GROUP-002: Constitution Snapshot Commands

Purpose:

- Support source-driven constitution snapshot generation.
- Write and validate `current.index.json`.
- Expose reload/load command handling for local Hermes sessions.
- Detect malformed snapshot block markers, including stray close markers.

Local files:

```text
hermes_cli/constitution_commands.py
hermes_cli/commands.py
hermes_cli/cli_commands_mixin.py
cli.py
tests/hermes_cli/test_commands.py
tests/test_constitution_commands_extended.py
```

Related constitution records:

- `docs/constitution-snapshot.md`
- `docs/session-startup-policy.md`

Review status:

```yaml
codex_review: approved_with_notes
risk_after_review: low
```

### PATCH-GROUP-003: Simple Shell Direct Mode

Purpose:

- Allow safe high-frequency read-only shell inspection without provider calls.
- Return raw stdout for allowlisted read-only commands.
- Reject shell metacharacters, write-capable command options, and credential or
  secret-looking paths.
- Avoid misclassifying Chinese natural-language task instructions containing
  punctuation such as angle brackets.

Local files:

```text
agent/shell_direct_mode.py
agent/conversation_loop.py
tests/agent/test_shell_direct_mode.py
```

Related constitution records:

- `docs/simple-shell-direct-mode.md`
- `docs/command-handler-design.md`

Review status:

```yaml
codex_review: approved_with_notes
risk_after_review: low
known_follow_up:
  - keep direct-mode allowlist narrow
  - continue treating credential-like paths as rejected
```

### PATCH-GROUP-004: Context Budget Helper

Purpose:

- Estimate local background-model text packet size without calling an LLM or a
  tokenizer service.
- Route text packets through `allow`, `needs_primary_approval`, or
  `reject_return_to_primary` decisions before any Ollama call.
- Keep the estimate deterministic, local-only, and stdlib-only.

Local files:

```text
agent/context_budget.py
tests/agent/test_context_budget.py
```

Related constitution records:

- `docs/background-local-model-adapter.md`
- `docs/validation/context-budget-helper-process-incident.md`
- `docs/validation/ollama-background-text-adapter-smoke-test.md`

Review status:

```yaml
codex_review: pass_with_notes_after_operator_judgment
risk_after_review: low
accepted_note: heuristic_not_tokenizer_truth
```

### PATCH-GROUP-005: Ollama Background Text Adapter

Purpose:

- Use local Ollama for bounded text preprocessing only.
- Enforce `output_authority: none`.
- Reject authority-like input before calling Ollama.
- Mark authority-like output and route it back to Hermes Primary.
- Bypass workstation HTTP proxies for local `127.0.0.1:11434` traffic.

Local files:

```text
agent/ollama_text_adapter.py
tests/agent/test_ollama_text_adapter.py
```

Related constitution records:

- `docs/background-local-model-adapter.md`
- `docs/validation/ollama-background-text-adapter-smoke-test.md`
- `docs/validation/ollama-process-boundary-incident.md`

Review status:

```yaml
codex_review: pass
risk_after_review: low
tests_reported: 56
```

### PATCH-GROUP-006: Ollama Dispatch Hook

Purpose:

- Add an explicit-purpose dispatch hook after Simple Shell Direct Mode and
  before turn-context construction.
- Keep the feature disabled by default.
- Preserve the original user message in every path.
- Inject only non-authoritative background text drafts when the adapter allows
  it.
- Suppress draft injection when the adapter routes back to Primary.

Local files:

```text
agent/conversation_loop.py
tests/agent/test_ollama_dispatch_integration.py
```

Environment gates:

```text
HERMES_OLLAMA_BACKGROUND_TEXT=1
```

Review status:

```yaml
codex_review: pass
risk_after_review: low
tests_reported: 76
```

### PATCH-GROUP-007: Ollama Deterministic Auto Classifier

Purpose:

- Reduce manual marker usage for low-risk text preprocessing.
- Use local deterministic classification only; no LLM decides whether to call
  Ollama.
- Route only high-confidence text-processing intents to Ollama.
- Check forbidden authority, execution, code-edit, provider-routing, file-path,
  shell, auth, dependency, database, and infrastructure signals before any
  allowed-pattern match.

Local files:

```text
agent/ollama_auto_classifier.py
agent/conversation_loop.py
tests/agent/test_ollama_auto_classifier.py
tests/agent/test_ollama_dispatch_integration.py
```

Environment gates:

```text
HERMES_OLLAMA_BACKGROUND_TEXT=1
HERMES_OLLAMA_BACKGROUND_TEXT_AUTO=1
```

Allowed automatic purposes:

```text
evidence_summary
translation
compression
text_normalization
```

Explicit marker remains required for broader purposes such as
`kanban_comment_draft`.

Review status:

```yaml
codex_review: pass
risk_after_review: low
tests_reported: 49
```

## Validation Summary

```yaml
local_patch_status: approved_with_notes
blocking_findings: 0
risk_after_review: low
ollama_series:
  OLLAMA-003: pass
  OLLAMA-004A: pass
  OLLAMA-004B: pass
  OLLAMA-004C: pass
estimated_deepseek_tokens_saved_in_004a: 4700
```

## Restore Guidance

The local restore manifest lives outside this repository:

```text
~/.hermes/local-patches/MANIFEST.md
```

Restore rule:

- Restore only after explicit operator approval.
- Check upstream file changes before copying local files back.
- If upstream changed the same file, do not overwrite blindly; enter review.
- Re-run relevant tests after restoring.
- Do not commit or push these local runtime patches to Hermes Agent upstream.

## Public Reuse Guidance

Other users may reuse part of this work, but they should treat each patch group
as independent:

- Take Simple Shell Direct Mode without Ollama if they only need read-only shell
  passthrough.
- Take Gateway Entry Guard without Weixin-specific operational assumptions.
- Take Ollama background processing only if they accept the local model
  authority boundary and context-budget gate.
- Re-run tests and review on their own Hermes Agent version.

The constitution policies in `docs/` are normative for this repository. The
runtime patch groups in this document are examples of one local implementation
that satisfied those policies in the operator's environment.
