---
name: gh-ai
description: AI-native GitHub CLI for LLM agents — token-optimized output (stripped JSON, minimal YAML, numbered lines), macro actions (create PR in one command), and error log extraction. Use instead of `gh` CLI or raw GitHub API calls when context window efficiency matters.
triggers:
  keywords:
    - GitHub API
    - repo tree
    - file read
    - issue view
    - PR view
    - PR checks
    - CI errors
    - action logs
    - apply patch
    - create PR
    - rate limit
    - search repos
    - search issues
    - gh-ai
  context:
    - User wants to fetch GitHub data without raw JSON bloat
    - User needs token-efficient GitHub interaction for LLM agent consumption
    - User wants to read repo files with line numbers
    - User wants to view issues/PRs with comments in minimal YAML
    - User wants to extract error blocks from CI logs
    - User wants to apply patches or create PRs in one step
---

# gh-ai — AI-Native GitHub CLI

Token-optimized GitHub CLI at `/home/peter/.local/bin/gh-ai`. Designed specifically for LLM agent consumption — strips all URLs, node IDs, avatars, and metadata from GitHub API responses. Returns minimal YAML, flat text, or numbered lines. Combines multi-step Git/GitHub operations into single macro commands.

## Prerequisites

- `GITHUB_TOKEN` env var set (or `GITHUB_PERSONAL_ACCESS_TOKEN`, `GH_TOKEN`)
- Token can be obtained via `export GITHUB_TOKEN=$(gh auth token)` if `gh` CLI is authenticated

## Commands

### Information Retrieval

```bash
# Repository file tree — filtered for code files only
gh-ai repo-tree owner/repo
gh-ai repo-tree owner/repo --max 100   # cap output (default 200)

# Read file with line numbers
gh-ai file-read owner/repo path/to/file.py
gh-ai file-read owner/repo path/to/file.py --start 10 --end 50

# View issue with all comments in compact YAML
gh-ai issue-view owner/repo 42

# View PR with diff stats, mergeable state, and reviews
gh-ai pr-view owner/repo 123

# Show CI check statuses for a PR
gh-ai pr-checks owner/repo 123

# Search repositories or issues (top 5 results)
gh-ai search "machine learning framework" --type repos
gh-ai search "segfault" --type issues

# Check API rate limit status
gh-ai rate-limit
```

### CI/CD and Debugging

```bash
# Extract error blocks from GitHub Actions logs (±10 lines context)
gh-ai get-action-errors owner/repo 1234567890
```

### Write Operations

```bash
# Create a new GitHub repo and push local content as the initial commit
gh-ai repo-init my-new-repo --description "A new project"
gh-ai repo-init my-repo --private                  # Private repo
gh-ai repo-init my-repo --org my-org               # Under organization
gh-ai repo-init my-repo --dry-run                  # Preview without creating

# Apply a unified diff from stdin
cat patch.diff | gh-ai apply-patch path/to/file.py

# Apply a search/replace block from stdin
cat <<'EOF' | gh-ai apply-patch path/to/file.py
<<<<<<< ORIGINAL
old code here
=======
new code here
>>>>>>> REPLACEMENT
EOF

# Create a PR — macro that branches, commits, pushes, and opens the PR
gh-ai create-pr owner/repo main feature-branch \
  --title "Fix authentication bug" \
  --body "Fixes the race condition in token refresh"

# Dry-run: show what would happen without pushing
gh-ai create-pr owner/repo main feature-branch \
  --title "Test" --body "Testing" --dry-run

# Create as draft PR
gh-ai create-pr owner/repo main feature-branch \
  --title "WIP: Refactor" --body "Still in progress" --draft
```

## Token Efficiency Rules

All output follows these rules:
- No `url`, `html_url`, `node_id`, `avatar_url`, `gravatar_id` fields
- No `followers_url`, `repos_url`, `subscriptions_url`, etc.
- Reactions condensed to `+1:4, heart:6` format (omitted if zero)
- Labels as comma-separated string, not array
- File output max 500 lines, log output max 500 lines
- Binary/media files filtered from `repo-tree`
- `file-read` always prepends line numbers (`42: def foo():`)

## Error Handling

| Error | Output |
|-------|--------|
| No token | `GITHUB_TOKEN (or GITHUB_PERSONAL_ACCESS_TOKEN, GH_TOKEN) not set` |
| 401 | `Bad credentials. Check GITHUB_TOKEN` |
| 403 rate limit | `Rate limited. Resets at {timestamp}` |
| 403 other | `Forbidden (403): {details}` |
| 404 | `Not found: may be private or doesn't exist` |
| Network error | Retries once (1s delay), then `Request failed: {error}` |

## Routing Rules

- **Reading code from a repo:** `file-read` with `--start`/`--end` to limit context
- **Exploring repo structure:** `repo-tree` to list files, then `file-read` specific ones
- **Investigating an issue:** `issue-view` gets issue + all comments in one call
- **Reviewing a PR:** `pr-view` for metadata + reviews, `pr-checks` for CI status
- **Debugging CI failures:** `pr-checks` to identify failed checks, then `get-action-errors` for error details
- **Before making API calls:** `rate-limit` to check remaining quota
- **Creating a PR:** `create-pr` handles the full pipeline; use `--dry-run` first to verify
- **Applying code changes:** `apply-patch` with either diff or search/replace format

## Anti-patterns

### Don't use gh or raw curl for token-sensitive operations

`gh issue view` returns human-formatted text with borders. `curl` on the API returns raw JSON with hundreds of URL fields. `gh-ai` gives minimal structured output designed for context windows.

### Don't skip rate-limit before batch operations

If making multiple API calls, check `gh-ai rate-limit` first. The tool doesn't auto-throttle — it's the caller's responsibility.

### Don't pipe binary content to apply-patch

`apply-patch` expects text patches (unified diff or search/replace blocks). It auto-detects the format.

### Don't use create-pr for repos without push access

The push step will fail with a clear git error. The tool checks for local changes first and aborts cleanly.
