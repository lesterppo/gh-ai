# gh-ai — AI-Native GitHub CLI

Token-optimized GitHub CLI designed for LLM agent consumption. Standard GitHub APIs return massive, deeply nested JSON — gh-ai strips all noise and combines multi-step operations into single macro commands.

## Why

| Approach | Token Cost (typical PR view) |
|----------|------------------------------|
| Raw GitHub API | ~8,000 tokens (URLs, node IDs, avatars, metadata) |
| `gh` CLI | ~3,000 tokens (table borders, human formatting) |
| **gh-ai** | **~400 tokens** (minimal YAML, no URLs, no metadata) |

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
gh-ai file-read owner/repo path.py      # Numbered lines
gh-ai issue-view owner/repo 42          # Issue + comments as YAML
gh-ai pr-view owner/repo 123            # PR + diff stats + reviews
gh-ai pr-checks owner/repo 123          # CI check statuses
gh-ai search "query" --type repos       # Top 5 results
gh-ai rate-limit                        # API quota
```

### Debug

```bash
gh-ai get-action-errors owner/repo RUN_ID   # Error blocks ±10 lines context
```

### Write

```bash
cat diff.patch | gh-ai apply-patch file.py       # Unified diff or search/replace
gh-ai create-pr owner/repo main branch \         # Full PR pipeline
  --title "Fix" --body "Details"
```

## Token Efficiency Rules

- No `url`, `html_url`, `node_id`, `avatar_url`, `gravatar_id`
- Reactions: `+1:4, heart:6` (omitted if zero)
- Labels as comma-separated string
- Max 500 lines per output
- Binary/media paths filtered from repo-tree

## Skills

- `skills/claude/SKILL.md` — Claude Code skill
- `skills/hermes/SKILL.md` — Hermes Agent skill
