# gh-ai — AI-Native GitHub CLI

Token-optimized GitHub CLI designed for LLM agent consumption. Standard GitHub APIs return massive, deeply nested JSON — gh-ai strips all noise and combines multi-step operations into single macro commands.

## Why

Measured character counts on identical queries (fewer chars = fewer tokens):

| Operation | `gh` CLI (`--json`) | `gh-ai` | Reduction |
|-----------|---------------------|---------|-----------|
| PR view | 667 | 378 | **1.8x** |
| Issue + comments (5+) | 187,941 | 28,172 | **6.7x** |
| Search repos (top 5) | 1,607 | 1,348 | **1.2x** |
| Rate limit | 1,146 | 86 | **13.3x** |
| Repo tree (flat) | 7 | 7 | ~1x |

Tested against `python/cpython` and `octocat/Hello-World` public repos. Against raw GitHub API (no `--json` filtering), gh-ai reductions exceed **50-200x** by stripping all URL, node ID, avatar, and metadata fields.

## Install

```bash
cp gh-ai ~/.local/bin/
chmod +x ~/.local/bin/gh-ai
```

Requires Python 3, `click`, `requests`, `PyYAML`. Set `GITHUB_TOKEN` (or `GITHUB_PERSONAL_ACCESS_TOKEN`, `GH_TOKEN`).

## Commands

### Read

```bash
gh-ai repo-tree owner/repo              # Filtered file tree
gh-ai repo-tree owner/repo --max 100    # Cap output
gh-ai file-read owner/repo path.py      # Numbered lines
gh-ai file-read owner/repo path.py --start 10 --end 50
gh-ai issue-view owner/repo 42          # Issue + comments as YAML
gh-ai pr-view owner/repo 123            # PR + diff stats + reviews
gh-ai pr-checks owner/repo 123          # CI check statuses
gh-ai pr-diff owner/repo 123            # PR diff with line numbers
gh-ai pr-diff owner/repo 123 --max-lines 100
gh-ai pr-status owner/repo 123          # One-line: state, mergeable, CI
gh-ai search "query" --type repos       # Top 5 results
gh-ai search "query" --type issues      # Top 5 issues
gh-ai rate-limit                        # API quota
```

### Debug

```bash
gh-ai get-action-errors owner/repo RUN_ID   # Error blocks ±10 lines context
```

### Write

```bash
# Create a repo and push initial content
gh-ai repo-init my-repo --description "A new project"
gh-ai repo-init my-repo --private
gh-ai repo-init my-repo --org my-org

# Create a PR (body from --body, --body-file, or stdin)
gh-ai create-pr owner/repo main branch --title "Fix" --body "Details"
echo "Body text" | gh-ai create-pr owner/repo main branch --title "Fix"
gh-ai create-pr owner/repo main branch --title "Fix" --body-file body.md

# Merge a PR
gh-ai pr-merge owner/repo 42
gh-ai pr-merge owner/repo 42 --squash --delete-branch
gh-ai pr-merge owner/repo 42 --rebase

# Comment on an issue or PR
echo "LGTM" | gh-ai pr-comment owner/repo 42
gh-ai pr-comment owner/repo 42 --body "Nice work"

# Create an issue
gh-ai issue-create owner/repo --title "Bug" --body "Details"
gh-ai issue-create owner/repo --title "Bug" --label bug --label high-priority

# Apply patches
cat diff.patch | gh-ai apply-patch file.py
```

## Token Efficiency Rules

- No `url`, `html_url`, `node_id`, `avatar_url`, `gravatar_id`
- Reactions: `+1:4, heart:6` (omitted if zero)
- Labels as comma-separated string
- Max 500 lines per output
- Binary/media paths filtered from repo-tree
- Line numbers on all file/diff output

## Skills

- `skills/claude/SKILL.md` — Claude Code skill
- `skills/hermes/SKILL.md` — Hermes Agent skill
