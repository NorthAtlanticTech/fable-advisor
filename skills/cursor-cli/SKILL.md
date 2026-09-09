---
name: cursor-cli
description: Operator's reference for driving the Cursor CLI (`agent`) headlessly — canonical print-mode invocation, verified Grok/model slugs, the failure signatures that exit 0 while doing nothing, sandbox and workspace-trust requirements, inherited CURSOR_API_KEY/stored-login auth, and session resume. USE WHEN invoking or debugging the `agent` / `cursor-agent` CLI, running the grok-implementer lane, seeing an EPERM on ~/.cursor/cli-config.json, a "Workspace Trust Required" prompt, certificate trust-settings noise, an empty agent run, or a lane that reported success without changing any files.
---

# Cursor CLI — headless operator's card

The `agent` binary ([cursor.com/cli](https://cursor.com/cli)) is how the `grok-implementer` lane produces code. Print mode (`-p`) is a one-shot headless run. Everything below is verified against the CLI, not inferred.

## Canonical invocation

Run the wrapping Bash call **with the host command sandbox disabled** (`dangerouslyDisableSandbox` in Claude Code) — see Sandbox below.

```bash
AGENT=$(command -v agent || command -v cursor-agent || true)
MODEL=cursor-grok-4.6-high-fast
SPEC=$(mktemp -t agent-spec.XXXXXX)
FINAL=$(mktemp -t agent-final.XXXXXX)

cat > "$SPEC" << 'SPEC_EOF'
[objective, files, interfaces, constraints, verification —
end with: "Run the verification command and include its actual output."]
SPEC_EOF

# Portable timeout: macOS has no `timeout` unless coreutils is installed
T=$(command -v gtimeout || command -v timeout || true)

${T:+$T 600} "$AGENT" -p "$(cat "$SPEC")" \
  --model "$MODEL" \
  --force \
  --trust \
  --output-format text \
  --workspace "$(pwd)" \
  > "$FINAL" 2>&1
```

| Flag | Why |
|---|---|
| `-p` | Headless print mode. Full tool access, including write and shell. |
| `--model` | Pin the producer. **Verify the slug first** (below). |
| `--force` | Apply edits and run commands unattended. Without it, print-mode edits are only *proposed*. |
| `--trust` | Trust the workspace without prompting — headless-only. Omit it in an untrusted directory and the run stalls on a prompt nobody can answer. |
| `--output-format` | `text` (default) \| `json` \| `stream-json`. |
| `--workspace` | Deterministic working root. `--add-dir` adds extra roots. |
| `--sandbox enabled\|disabled` | *Cursor's own* sandbox for the commands the agent runs — unrelated to your host agent's sandbox. |

## Model slugs — verify, never assume

```bash
"$AGENT" --list-models | grep -q "^${MODEL} " || echo "ABORT: $MODEL not available"
```

**The CLI silently accepts unknown slugs and answers anyway**, falling back to some other model with no error — a typo swaps the producer and defeats cross-vendor routing invisibly. There is no error to catch; the `--list-models` check is the only detection.

Grok 4.6 supports `cursor-grok-4.6-low`, `cursor-grok-4.6-medium`, `cursor-grok-4.6-high` (the named-model default), and `cursor-grok-4.6-xhigh`; Fast variants add `-fast`. This plugin pins `cursor-grok-4.6-high-fast`. Account, plan, and team-policy availability can differ, so the live `--list-models` output remains authoritative.

## Exit codes lie

`agent` exits **0** on at least three failure modes that changed nothing. Assert on artifacts — a non-empty `$FINAL` describing real work, plus `git status` / `git diff` showing edits — never on `$?`.

| Signature | Cause | Fix |
|---|---|---|
| `EPERM: operation not permitted, open '~/.cursor/cli-config.json.<pid>.<uuid>.tmp'` | Host sandbox blocked the startup config rewrite that `--force` triggers | Re-run unsandboxed, or allowlist writes to `~/.cursor`. **Not** an auth problem. |
| `Workspace Trust Required` prompt text | Untrusted directory, unanswerable in `-p` | Add `--trust` |
| Empty `$FINAL` | Run produced nothing | Treat as failure; report raw output |
| *(no signature at all)* | Unrecognized `--model` slug → silent fallback | Preflight `--list-models` |

Benign noise, do not diagnose from it: `ERROR: failed to copy trust settings of system certificate-…` on stderr under a sandbox. TLS still works (`--list-models` succeeds alongside it). Filter it before reporting.

## Auth

The CLI accepts either an inherited `CURSOR_API_KEY` or a stored interactive login. For headless sessions and CI, Cursor recommends the environment variable:

```bash
test -n "${CURSOR_API_KEY:-}" || echo "CURSOR_API_KEY is not set; checking stored login"
"$AGENT" status   # run unsandboxed; validates whichever auth source is available
```

- Supply `CURSOR_API_KEY` to the environment that launches the host agent so the child Cursor CLI inherits it. An `.env` file is not loaded automatically unless the shell, launcher, or CI step explicitly loads it. Restart an already-running host after installing the CLI or adding the variable so it inherits the new `PATH` and environment.
- Never echo the value, put it in the agent prompt, save it in a temporary file, include it in reports, or pass it directly as `--api-key <key>` where shell history and process listings may expose it.
- If `CURSOR_API_KEY` is present, do not run `agent login`; use `status` and `--list-models` to validate access.
- Without the variable, Cursor can use credentials saved by `agent login` (the macOS CLI stores them in Keychain). Run `agent login` only after `status` actually reports logged-out.
- `agent logout` clears stored auth — never run it to "reset" a failing run.

Generate user API keys under **Integrations → User API Keys** in the Cursor dashboard. See Cursor's [CLI authentication guide](https://docs.cursor.com/en/cli/reference/authentication).

## Session resume

A timed-out or killed run does not lose its work:

```bash
"$AGENT" ls                 # list recent chat sessions with IDs
"$AGENT" --resume <chatId>  # resume a specific session
"$AGENT" --continue         # continue the most recent one
```

Resume once with a corrected or narrowed prompt; if it fails again, report what landed rather than restarting from zero.

## Verify independently

The agent's final message is a claim, not evidence. Read `git diff`, re-run the spec's verification command yourself, and compare its real output against what the agent said. `--force` let it run commands without review — that is exactly why independent re-verification is mandatory.
