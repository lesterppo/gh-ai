# gh-ai — AI-Native GitHub CLI

Token-optimized GitHub CLI designed for LLM agent consumption. Standard GitHub APIs return massive, deeply nested JSON — gh-ai strips all noise and combines multi-step operations into single macro commands.

Covers the **full GitHub REST API surface** with 92 commands: repositories, contents, commits, branches, issues, pull requests, GitHub Actions, search, users/orgs, gists, releases, notifications, and a raw API passthrough.

## Why

Measured character counts on identical queries (fewer chars = fewer tokens):

| Operation | `gh` CLI / raw API | `gh-ai` | Reduction |
| --- | --- | --- | --- |
| PR view | 667 | 378 | **1.8x** |
| Issue + comments (5+) | 187,941 | 28,172 | **6.7x** |
| Search repos (top 5) | 1,607 | 1,348 | **1.2x** |
| Rate limit | 1,146 | 130 | **8.8x** |
| Repo view | 323 | 304 | ~1x (readable) |
| Commit list (5) | 590 | 364 | **1.6x** |

Against raw GitHub API responses (no `--json` filtering), gh-ai reductions exceed **50-200x** by stripping all URL, node ID, avatar, and metadata fields.

## Install

```
cp gh-ai ~/.local/bin/
chmod +x ~/.local/bin/gh-ai
```

Requires Python 3, `click`, `requests`, `PyYAML`. Set `GITHUB_TOKEN` (or `GITHUB_PERSONAL_ACCESS_TOKEN`, `GH_TOKEN`).

```bash
# If gh CLI is authenticated:
export GITHUB_TOKEN=$(gh auth token)
```

### Token scopes

| Scope | Needed for |
| --- | --- |
| `repo` | Most read/write operations on repos |
| `workflow` | Writing/triggering GitHub Actions workflows |
| `delete_repo` | `repo-delete` |
| `gist` | `gist-create` / `gist-list` / `gist-delete` |
| `notifications` | `notif-list` / `notif-read` |
| `read:org` | Private org members/teams |

## Command Catalog

### Info & utils

```
gh-ai rate-limit                          # API quota status
gh-ai auth-status                         # Login, scopes, rate
gh-ai meta                                # API metadata/features
gh-ai license-list                        # License templates
gh-ai gitignore-list                      # .gitignore templates
gh-ai gitignore-get python                # Template source
gh-ai api /repos/o/r --method PATCH --data '{"description":"x"}'
                                          # Raw REST passthrough (any endpoint)
```

### Repositories

```
gh-ai repo-view owner/repo                # Compact repo info
gh-ai repo-list [--owner u] [--type owner|member|public|private] [--sort created]
gh-ai repo-init name [--private] [--org org] [--description "..."] [--dry-run]
                                          # Create repo + push initial commit
gh-ai repo-clone owner/repo [dir] [--depth N]
gh-ai repo-fork owner/repo [--org org]
gh-ai repo-delete owner/repo --yes        # Irreversible
gh-ai repo-update owner/repo [--name x] [--description "..."] [--homepage URL]
                          [--private|--public] [--default-branch main]
gh-ai repo-topics owner/repo [--set a --set b | --add a --remove b]
gh-ai repo-langs owner/repo               # Language byte breakdown
gh-ai repo-contributors owner/repo        # Top contributors
gh-ai repo-branches owner/repo
gh-ai repo-tags owner/repo
gh-ai repo-license owner/repo
gh-ai repo-archive owner/repo --enable|--disable
```

### Contents

```
gh-ai repo-tree owner/repo [--max 200]    # Code-only file tree
gh-ai dir-list owner/repo [path] [--ref branch]
gh-ai file-read owner/repo path [--start 10] [--end 50]
gh-ai file-write owner/repo path --content "..." [--file f] [--message "commit"] [--branch b] [--sha ...]
                                          # --sha required to update an existing file
gh-ai file-delete owner/repo path --message "commit" [--branch b] [--sha ...]
gh-ai readme owner/repo [--ref branch]
```

### Commits & branches

```
gh-ai commit-list owner/repo [--branch b] [--path f] [--author u] [--max 20]
gh-ai commit-view owner/repo SHA          # Message, stats, files, status
gh-ai commit-compare owner/repo base...head
gh-ai commit-search "query"
gh-ai branch-create owner/repo name [--sha x] [--from branch]
gh-ai branch-delete owner/repo name --yes
```

### Issues

```
gh-ai issue-list owner/repo [--state open|closed|all] [--label bug] [--assignee u] [--author u]
gh-ai issue-view owner/repo N             # Issue + comments as YAML
gh-ai issue-create owner/repo --title "..." [--label bug] [--assignee u]
gh-ai issue-update owner/repo N [--title x] [--body x] [--state open|closed]
                       [--label a --label b | --add-label x --remove-label y]
                       [--assignee u | --add-assignee u --remove-assignee u] [--milestone 1]
gh-ai issue-close owner/repo N
gh-ai issue-reopen owner/repo N
gh-ai issue-labels owner/repo
gh-ai issue-milestones owner/repo [--state open]
gh-ai issue-milestone-create owner/repo "v1.0" [--description "..."] [--due-on 2026-12-31]
gh-ai issue-events owner/repo N           # Timeline
```

### Pull requests

```
gh-ai pr-list owner/repo [--state open|closed|all] [--author u] [--label bug] [--base main] [--head user:branch]
gh-ai pr-view owner/repo N                # PR + diff stats + reviews as YAML
gh-ai pr-create owner/repo --title "..." --head feature --base main [--body "..."] [--draft]
gh-ai create-pr owner/repo main branch --title "..." --body "..." [--draft] [--dry-run]
                                          # Macro: commit + push + open PR
gh-ai pr-update owner/repo N [--title x] [--body x] [--state open|closed] [--base main]
gh-ai pr-close owner/repo N
gh-ai pr-merge owner/repo N [--squash|--rebase] [--delete-branch] [--commit-title x]
gh-ai pr-comment owner/repo N [--body "..." | --body-file f | stdin]
gh-ai pr-review owner/repo N --approve|--request-changes|--event comment [--body "..."]
gh-ai pr-review-comment owner/repo N --path file.py --line 42 --body "..."
gh-ai pr-comments owner/repo N            # Issue comments + inline + review summaries
gh-ai pr-files owner/repo N
gh-ai pr-reviewers owner/repo N [--add u --add v] [--remove u]
gh-ai pr-checks owner/repo N              # CI check statuses
gh-ai pr-diff owner/repo N [--max-lines 300]
gh-ai pr-status owner/repo N              # One-liner: state, mergeable, CI
```

### GitHub Actions

```
gh-ai action-list owner/repo               # Workflows
gh-ai action-runs owner/repo [--branch b] [--status success|failure|...] [--event push] [--workflow ci.yml]
gh-ai action-run owner/repo RUN_ID         # Run + jobs + steps
gh-ai action-jobs owner/repo RUN_ID
gh-ai action-trigger owner/repo ci.yml --ref main [--inputs '{"env":"prod"}']
gh-ai action-cancel owner/repo RUN_ID
gh-ai action-rerun owner/repo RUN_ID [--failed]
gh-ai get-action-errors owner/repo RUN_ID  # Error blocks ±10 lines from logs
gh-ai action-secret owner/repo [--set NAME --value v | --delete NAME]   # list by default
gh-ai action-variable owner/repo [--set NAME --value v | --delete NAME] # list by default
gh-ai action-cache owner/repo
```

### Search

```
gh-ai search "query" --type repos|issues|code|users|commits|topics [--max 5]
gh-ai search-code "query" [--repo owner/repo] [--max 5]
```

### Users, orgs, gists

```
gh-ai user-view [login]                   # Profile (default: self)
gh-ai user-repos [login] [--sort created]
gh-ai user-gists [login]
gh-ai org-view org
gh-ai org-members org [--role all|admin|member]
gh-ai org-repos org [--type all|public|private]
gh-ai org-teams org
gh-ai gist-list [--login u]
gh-ai gist-create --description "..." --file a.py --file b.py | --name n --content c | --stdin-name f
gh-ai gist-view GIST_ID
gh-ai gist-delete GIST_ID --yes
```

### Releases & notifications

```
gh-ai release-list owner/repo
gh-ai release-view owner/repo [TAG | --latest]
gh-ai release-create owner/repo v1.0.0 [--name x] [--body "..."] [--draft] [--prerelease] [--target SHA]
gh-ai release-upload owner/repo TAG file [--name asset]
gh-ai release-delete owner/repo TAG --yes
gh-ai notif-list [--all] [--participating]
gh-ai notif-read [--thread ID]            # Mark all (or one) read
```

### Patch application

```
cat diff.patch | gh-ai apply-patch file.py   # Unified diff or search/replace block
```

## Token Efficiency Rules

- No `url`, `html_url`, `node_id`, `avatar_url`, `gravatar_id`
- Reactions: `+1:4, heart:6` (omitted if zero)
- Labels as comma-separated string
- Max 500 lines per output; list commands default to 20 items (`--max` to change)
- Binary/media paths filtered from repo-tree
- Line numbers on all file/diff output

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

## Skills

- `skills/claude/SKILL.md` — Claude Code skill
- `skills/hermes/SKILL.md` — Hermes Agent skill

## License

MIT
