# gh-ai agent wrappers

`gh-ai-agent` — entry point for Muse agents authenticating via the Secure Vault.

The plain `gh-ai` script needs a `GITHUB_TOKEN` env var. This wrapper instead
fetches a fresh authd credential surrogate for the stored `custom.github`
connector on every invocation and exposes it as `GITHUB_TOKEN`. The surrogate
(`hsurr:*`) is swapped for the real credential by Sentinel/authd on approved
egress to `api.github.com` — the raw token never touches the agent's memory,
files, or logs.

## Install

```bash
cp wrappers/gh-ai-agent ~/.local/bin/gh-ai
chmod +x ~/.local/bin/gh-ai
```

## Configuration

All paths resolve with env overrides first, then repo-relative, then legacy
install paths:

| Var | Purpose | Fallbacks |
|-----|---------|-----------|
| `DYNAMIC_CREDENTIALS_DIR` | dir with `dynamic_credentials.py` | known skill install locations (probed, not hardcoded — fixed 2026-10-15 after a deployment moved the module and broke the old hardcoded path) |
| `GH_AI_REAL_BIN` | the real `gh-ai` CLI script | `<repo>/gh-ai`, `/home/hatch/.local/bin/gh-ai-real` |
| `GH_AI_VENV_PYTHON` | python with `click`/`requests`/`PyYAML` | `<repo>/.venv/bin/python`, legacy venv, `sys.executable` |

## Security notes

- Only `api.github.com` is allow-listed for the surrogate (`ensure_allowed_url`).
- `GITHUB_PERSONAL_ACCESS_TOKEN` / `GH_TOKEN` are stripped from the child env so
  the surrogate is the single credential in play.
- A non-`hsurr:` response from authd aborts instead of proceeding tokenless.
