---
name: gh-ai
description: AI-native GitHub CLI for LLM agents — 92 token-optimized commands covering the full GitHub REST surface. Use instead of `gh` or raw API for token-efficient GitHub operations.
version: 2.0.0
author: Claude Code + Peter
license: MIT
platforms: [linux]
metadata:
  hermes:
    tags: [GitHub, CLI, Token-Efficiency, CI/CD, PR-Automation, Patch-Application]
    related_skills: [github, github-auth, github-pr-workflow, github-code-review]
---

# gh-ai — AI-Native GitHub CLI

Token-optimized GitHub CLI at `/home/peter/.local/bin/gh-ai` (v2.0.0). Strips all URLs, node IDs, avatars, and metadata from GitHub API responses. Combines multi-step Git/GitHub operations into single macro commands. Covers the full GitHub REST API surface: repos, contents, commits, branches, issues, PRs, Actions, search, users/orgs, gists, releases, notifications.

## Prerequisites

- `GITHUB_TOKEN` env var (or `GITHUB_PERSONAL_ACCESS_TOKEN`, `GH_TOKEN`)
- If `gh` CLI is authenticated, use: `export GITHUB_TOKEN=$(gh auth token)`
- `pynacl` only needed for `action-secret --set`

## Commands (92 total)

### Info / utils
```bash
gh-ai rate-limit | auth-status | meta | license-list | gitignore-list | gitignore-get T
gh-ai api /repos/o/r --method PATCH --data '{"description":"x"}'   # raw passthrough
```

### Repositories
```bash
gh-ai repo-view owner/repo | repo-list [--owner u] | repo-init name [--private] [--org o]
gh-ai repo-clone owner/repo [dir] | repo-fork owner/repo | repo-delete owner/repo --yes
gh-ai repo-update owner/repo [--name x] [--description "..."] [--private|--public]
gh-ai repo-topics owner/repo [--set a --set b | --add a --remove b]
gh-ai repo-langs | repo-contributors | repo-branches | repo-tags | repo-license
gh-ai repo-archive owner/repo --enable|--disable
```

### Contents
```bash
gh-ai repo-tree owner/repo [--max 200] | dir-list owner/repo [path]
gh-ai file-read owner/repo path [--start N] [--end N]
gh-ai file-write owner/repo path --content "..." [--file f] --message "..." [--branch b] [--sha ...]
gh-ai file-delete owner/repo path --message "..." [--branch b] | readme owner/repo
```

### Commits & branches
```bash
gh-ai commit-list [--branch b] [--path f] | commit-view SHA | commit-compare base...head
gh-ai commit-search "q" | branch-create name [--from b] | branch-delete name --yes
```

### Issues
```bash
gh-ai issue-list [--state open|closed|all] [--label bug] [--author u] [--max 20]
gh-ai issue-view N | issue-create --title "..." [--label bug]
gh-ai issue-update N [--title x] [--state open|closed] [--add-label x] [--remove-label y] [--assignee u]
gh-ai issue-close N | issue-reopen N | issue-labels | issue-milestones
gh-ai issue-milestone-create "v1.0" | issue-events N
```

### Pull requests
```bash
gh-ai pr-list [--state open] [--author u] [--label bug] [--base main] [--max 20]
gh-ai pr-view N | pr-create --title "..." --head feature --base main [--draft]
gh-ai create-pr owner/repo main branch --title "..." --body "..." [--dry-run]   # macro
gh-ai pr-update N [--title x] | pr-close N | pr-merge N [--squash|--rebase] [--delete-branch]
gh-ai pr-comment N [--body "..." | stdin] | pr-review N --approve|--request-changes [--body "..."]
gh-ai pr-review-comment N --path f --line 42 --body "..."
gh-ai pr-comments N | pr-files N | pr-reviewers [--add u] | pr-checks N | pr-diff N | pr-status N
```

### GitHub Actions
```bash
gh-ai action-list | action-runs [--branch b] [--status failure] [--max 20]
gh-ai action-run RUN_ID | action-jobs RUN_ID
gh-ai action-trigger ci.yml --ref main [--inputs '{"k":"v"}']
gh-ai action-cancel RUN_ID | action-rerun RUN_ID [--failed]
gh-ai get-action-errors RUN_ID            # error blocks ±10 lines
gh-ai action-secret [--set NAME --value v | --delete NAME] | action-variable [--set|--delete]
gh-ai action-cache
```

### Search
```bash
gh-ai search "q" --type repos|issues|code|users|commits|topics [--max 5]
gh-ai search-code "q" [--repo owner/repo]
```

### Users / orgs / gists / releases / notifications
```bash
gh-ai user-view [login] | user-repos [login] | user-gists [login]
gh-ai org-view org | org-members org | org-repos org | org-teams org
gh-ai gist-list | gist-create --description "..." --file a.py [--stdin-name f]
gh-ai gist-view ID | gist-delete ID --yes
gh-ai release-list | release-view [TAG|--latest] | release-create TAG [--draft] [--prerelease]
gh-ai release-upload TAG file | release-delete TAG --yes
gh-ai notif-list [--all] | notif-read [--thread ID]
```

### Patch application
```bash
cat diff.patch | gh-ai apply-patch file.py     # unified diff or search/replace
```

## Output Rules

- No URLs, node IDs, avatars, gravatars in output
- Reactions: `+1:4, heart:6` (omitted if zero)
- Labels: comma-separated string
- File/log output capped at 500 lines; lists default 20 (`--max` to change)
- Binary/media paths filtered from repo-tree
- Line numbers on all file reads: `42: def foo():`

## Error Codes

| Exit | Meaning |
|------|---------|
| 1 | Input error (no token, no stdin for patch, missing confirm) |
| 2 | Network error after retry |
| 3 | 401 — bad credentials |
| 4 | 403 — rate limited or forbidden |
| 5 | 404 — not found |
| 6 | Other HTTP error (422 validation, etc.) |
| 7 | Git command failed |
| 8 | Git command timed out |
| 9 | Git not found on PATH |

## Notes

- Destructive commands (`repo-delete`, `branch-delete`, `gist-delete`, `release-delete`) require `--yes`
- `file-write` needs `--sha` to update an existing file (GET it first or let `file-delete` auto-fetch)
- `/user/gists` may 404 for some tokens even with gist scope (GitHub quirk) — `gist-create` still works
- Code search results can be empty for recently pushed repos (index lag, `incomplete_results`)
