---
name: gh-ai
description: AI-native GitHub CLI for LLM agents — 92 token-optimized commands covering the full GitHub REST surface (repos, issues, PRs, Actions, search, gists, releases, notifications, raw API). Use instead of `gh` CLI or raw GitHub API calls when context window efficiency matters.
triggers:
  keywords:
    - GitHub API
    - repo tree
    - file read
    - issue view
    - issue list
    - PR view
    - PR checks
    - PR review
    - CI errors
    - action logs
    - action trigger
    - apply patch
    - create PR
    - rate limit
    - search repos
    - search code
    - repo list
    - release
    - gist
    - gh-ai
  context:
    - User wants to fetch GitHub data without raw JSON bloat
    - User needs token-efficient GitHub interaction for LLM agent consumption
    - User wants to read repo files with line numbers
    - User wants to view/list issues or PRs in compact form
    - User wants to approve or comment on a PR
    - User wants to extract error blocks from CI logs or trigger workflows
    - User wants to apply patches or create PRs in one step
    - User wants to search code, users, commits, or topics
---

# gh-ai — AI-Native GitHub CLI

Token-optimized GitHub CLI at `/home/peter/.local/bin/gh-ai` (v2.0.0, 92 commands). Designed specifically for LLM agent consumption — strips all URLs, node IDs, avatars, and metadata from GitHub API responses. Returns compact lines, minimal YAML, or numbered lines. Combines multi-step Git/GitHub operations into single macro commands.

## Prerequisites

- `GITHUB_TOKEN` env var set (or `GITHUB_PERSONAL_ACCESS_TOKEN`, `GH_TOKEN`)
- Token can be obtained via `export GITHUB_TOKEN=$(gh auth token)` if `gh` CLI is authenticated
- `pynacl` needed only for `action-secret --set` (libsodium encryption)

## Commands

### Info & utils

```bash
gh-ai rate-limit                          # API quota status
gh-ai auth-status                         # Login, scopes, rate
gh-ai api /repos/o/r --method PATCH --data '{"description":"x"}'   # raw passthrough
```

### Repositories

```bash
gh-ai repo-view owner/repo                # compact info
gh-ai repo-list [--owner u] [--max 20]    # list repos
gh-ai repo-init name [--private] [--org org] [--dry-run]   # create + push
gh-ai repo-fork owner/repo [--org org]
gh-ai repo-delete owner/repo --yes
gh-ai repo-update owner/repo [--description "..."] [--private|--public]
gh-ai repo-topics owner/repo [--set a --set b | --add a --remove b]
gh-ai repo-langs / repo-contributors / repo-branches / repo-tags / repo-license
gh-ai repo-archive owner/repo --enable|--disable
```

### Contents

```bash
gh-ai repo-tree owner/repo [--max 200]
gh-ai dir-list owner/repo [path] [--ref branch]
gh-ai file-read owner/repo path [--start 10] [--end 50]
gh-ai file-write owner/repo path --content "..." [--file f] --message "..." [--branch b] [--sha ...]
gh-ai file-delete owner/repo path --message "..." [--branch b]
gh-ai readme owner/repo
```

### Commits & branches

```bash
gh-ai commit-list owner/repo [--branch b] [--path f] [--max 20]
gh-ai commit-view owner/repo SHA
gh-ai commit-compare owner/repo base...head
gh-ai commit-search "query"
gh-ai branch-create owner/repo name [--from branch]
gh-ai branch-delete owner/repo name --yes
```

### Issues

```bash
gh-ai issue-list owner/repo [--state open|closed|all] [--label bug] [--max 20]
gh-ai issue-view owner/repo N              # YAML + comments
gh-ai issue-create owner/repo --title "..." [--label bug] [--body "..."]
gh-ai issue-update owner/repo N [--title x] [--state open|closed] [--add-label x] [--remove-label y]
gh-ai issue-close owner/repo N / issue-reopen owner/repo N
gh-ai issue-labels / issue-milestones / issue-milestone-create / issue-events
```

### Pull requests

```bash
gh-ai pr-list owner/repo [--state open] [--author u] [--max 20]
gh-ai pr-view owner/repo N                # YAML + diff stats + reviews
gh-ai pr-create owner/repo --title "..." --head feature --base main [--draft]
gh-ai create-pr owner/repo main branch --title "..." --body "..." [--dry-run]  # macro
gh-ai pr-update / pr-close / pr-merge [--squash|--rebase] [--delete-branch]
gh-ai pr-comment owner/repo N [--body "..." | stdin]
gh-ai pr-review owner/repo N --approve|--request-changes [--body "..."]
gh-ai pr-review-comment owner/repo N --path f --line 42 --body "..."
gh-ai pr-comments / pr-files / pr-reviewers [--add u]
gh-ai pr-checks / pr-diff / pr-status
```

### GitHub Actions

```bash
gh-ai action-list owner/repo
gh-ai action-runs owner/repo [--branch b] [--status failure] [--max 20]
gh-ai action-run owner/repo RUN_ID        # run + jobs + steps
gh-ai action-trigger owner/repo ci.yml --ref main [--inputs '{"k":"v"}']
gh-ai action-cancel / action-rerun [--failed]
gh-ai get-action-errors owner/repo RUN_ID  # error blocks ±10 lines
gh-ai action-secret owner/repo [--set NAME --value v | --delete NAME]
gh-ai action-variable owner/repo [--set NAME --value v | --delete NAME]
```

### Search

```bash
gh-ai search "query" --type repos|issues|code|users|commits|topics [--max 5]
gh-ai search-code "query" [--repo owner/repo]
```

### Users, orgs, gists, releases, notifications

```bash
gh-ai user-view [login] / user-repos / user-gists
gh-ai org-view org / org-members / org-repos / org-teams
gh-ai gist-list / gist-create --description "..." --file a.py / gist-view ID / gist-delete ID --yes
gh-ai release-list / release-view TAG|--latest / release-create TAG / release-upload TAG file / release-delete TAG --yes
gh-ai notif-list / notif-read [--thread ID]
```

### Patch application

```bash
cat diff.patch | gh-ai apply-patch path/to/file.py          # unified diff
cat <<'EOF' | gh-ai apply-patch path/to/file.py             # search/replace
<<<<<<< ORIGINAL
old code here
=======
new code here
>>>>>>> REPLACEMENT
EOF
```

## Token Efficiency Rules

All output follows these rules:
- No `url`, `html_url`, `node_id`, `avatar_url`, `gravatar_id` fields
- No `followers_url`, `repos_url`, `subscriptions_url`, etc.
- Reactions condensed to `+1:4, heart:6` format (omitted if zero)
- Labels as comma-separated string, not array
- File output max 500 lines, log output max 500 lines, lists default 20
- Binary/media files filtered from `repo-tree`
- `file-read` always prepends line numbers (`42: def foo():`)

## Error Handling

| Exit | Meaning |
|------|---------|
| 1 | Input error (no token, no stdin for patch, missing confirm) |
| 2 | Network error after retry |
| 3 | 401 — bad credentials |
| 4 | 403 — rate limited or forbidden |
| 5 | 404 — not found |
| 6 | Other HTTP error (422 validation, etc.) |
| 7-9 | Git failures |

## Routing Rules

- **Reading code from a repo:** `file-read` with `--start`/`--end` to limit context
- **Exploring repo structure:** `repo-tree` to list files, then `file-read` specific ones
- **Investigating an issue:** `issue-view` gets issue + all comments in one call
- **Reviewing a PR:** `pr-view` for metadata + reviews, `pr-checks` for CI status, `pr-comments` for the full comment thread
- **Approving a PR:** `pr-review owner/repo N --approve --body "LGTM"`
- **Debugging CI failures:** `action-runs --status failure`, then `action-run` for jobs, then `get-action-errors` for error blocks
- **Triggering a workflow:** `action-trigger owner/repo file.yml --ref main --inputs '{...}'`
- **Before batch API calls:** `rate-limit` to check remaining quota
- **Creating a PR:** `create-pr` handles the full pipeline; use `--dry-run` first to verify
- **Applying code changes:** `apply-patch` with either diff or search/replace format
- **Anything not covered:** `api` raw passthrough to any REST endpoint

## Anti-patterns

### Don't use gh or raw curl for token-sensitive operations

`gh issue view` returns human-formatted text with borders. `curl` on the API returns raw JSON with hundreds of URL fields. `gh-ai` gives minimal structured output designed for context windows.

### Don't skip rate-limit before batch operations

If making multiple API calls, check `gh-ai rate-limit` first. The tool doesn't auto-throttle — it's the caller's responsibility.

### Don't pipe binary content to apply-patch

`apply-patch` expects text patches (unified diff or search/replace blocks). It auto-detects the format.

### Don't use create-pr for repos without push access

The push step will fail with a clear git error. The tool checks for local changes first and aborts cleanly.

### Don't forget --yes on destructive commands

`repo-delete`, `branch-delete`, `gist-delete`, `release-delete` all require an explicit `--yes` confirmation.
