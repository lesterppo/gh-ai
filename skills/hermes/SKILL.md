---
name: gh-ai
description: "AI-native GitHub CLI for LLM agents — token-optimized output, macro actions, error log extraction."
version: 1.0.0
author: Claude Code + Peter
license: MIT
platforms: [linux]
metadata:
  hermes:
    tags: [GitHub, CLI, Token-Efficiency, CI/CD, PR-Automation, Patch-Application]
    related_skills: [github-auth, github-pr-workflow, github-code-review]
---

# gh-ai — AI-Native GitHub CLI

Token-optimized GitHub CLI at `/home/peter/.local/bin/gh-ai`. Strips all URLs, node IDs, avatars, and metadata from GitHub API responses. Combines multi-step Git/GitHub operations into single macro commands.

## Prerequisites

- `GITHUB_TOKEN` env var (or `GITHUB_PERSONAL_ACCESS_TOKEN`, `GH_TOKEN`)
- If `gh` CLI is authenticated, use: `export GITHUB_TOKEN=$(gh auth token)`

## Commands

### Information Retrieval

```bash
gh-ai repo-tree owner/repo             # File tree, code files only
gh-ai repo-tree owner/repo --max 100   # Cap output
gh-ai file-read owner/repo path.py     # File with line numbers
gh-ai file-read owner/repo path.py --start 10 --end 50
gh-ai issue-view owner/repo 42         # Issue + comments as YAML
gh-ai pr-view owner/repo 123           # PR + diff stats + reviews
gh-ai pr-checks owner/repo 123         # CI check statuses
gh-ai search "query" --type repos      # Top 5 repos
gh-ai search "query" --type issues     # Top 5 issues
gh-ai rate-limit                       # API quota status
```

### CI/Debugging

```bash
gh-ai get-action-errors owner/repo RUN_ID   # Extract error blocks ±10 lines
```

### Write Operations

```bash
cat diff.patch | gh-ai apply-patch file.py   # Apply unified diff or search/replace

gh-ai create-pr owner/repo main branch \     # Full PR creation pipeline
  --title "Fix bug" --body "Details"

gh-ai create-pr owner/repo main branch \     # Dry-run first
  --title "Test" --body "Test" --dry-run
```

## Output Rules

- No URLs, node IDs, avatars, gravatars in output
- Reactions: `+1:4, heart:6` (omitted if zero)
- Labels: comma-separated string
- File/log output capped at 500 lines
- Binary/media paths filtered from repo-tree
- Line numbers on all file reads: `42: def foo():`

## Error Codes

| Exit | Meaning |
|------|---------|
| 1 | Input error (no token, no stdin for patch, clean working tree) |
| 2 | Network error after retry |
| 3 | 401 — bad credentials |
| 4 | 403 — rate limited or forbidden |
| 5 | 404 — not found |
| 6 | Other HTTP error |
| 7 | Git command failed |
| 8 | Git command timed out |
| 9 | Git not found on PATH |
