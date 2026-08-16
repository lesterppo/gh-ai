---
name: gh-ai
description: AI-native GitHub CLI for LLM agents — 172 token-optimized commands covering the full GitHub surface (REST + GraphQL: repos, contents, git, commits, branches, issues, PRs, Actions, Agents, Projects v2, Discussions, security, search, users/orgs/teams, gists, packages, wiki, releases, notifications).
version: 3.0.0
author: Claude Code + Peter
license: MIT
platforms: [linux]
metadata:
  hermes:
    tags: [GitHub, CLI, Token-Efficiency, CI/CD, PR-Automation, Projects, Discussions, Security]
    related_skills: [github, github-auth, github-pr-workflow, github-code-review]
---

# gh-ai — AI-Native GitHub CLI

Token-optimized GitHub CLI at `/home/peter/.local/bin/gh-ai` (v3.0.0, 172 commands). Strips all URLs, node IDs, avatars, and metadata from GitHub API responses. Combines multi-step Git/GitHub operations into single macro commands. Covers REST + GraphQL + git-protocol surface, mapped from a live crawl of the logged-in github.com web tree.

## Prerequisites

- `GITHUB_TOKEN` env var (or `GITHUB_PERSONAL_ACCESS_TOKEN`, `GH_TOKEN`); `export GITHUB_TOKEN=$(gh auth token)` works
- `pynacl` only for `action-secret --set` / `agent-secret --set`
- `project` scope for Projects v2 writes

## Commands (172 total)

### Info & utils
```bash
gh-ai rate-limit | auth-status | meta | zen | emojis | feeds
gh-ai license-list | gitignore-list | gitignore-get T | markdown "text"
gh-ai api /repos/o/r --method PATCH --data '{"description":"x"}'   # raw REST
gh-ai graphql 'query{viewer{login}}' [--var k=v]                    # raw GraphQL
```

### Repositories
```bash
gh-ai repo-view o/r | repo-list [--owner u] | repo-init name [--private] [--org o]
gh-ai repo-clone o/r [dir] | repo-fork o/r | repo-delete o/r --yes
gh-ai repo-update o/r [--name x] [--description] [--private|--public] [--default-branch b]
gh-ai repo-topics o/r [--set a --set b | --add a --remove b] | repo-langs o/r
gh-ai repo-contributors | repo-branches | repo-tags | repo-license | repo-archive --enable|--disable
gh-ai repo-transfer o/r NEW | repo-generate TEMPLATE name | repo-dl o/r [--zip] [--out f]
gh-ai repo-stats o/r --type contributors|commit-activity|code-frequency|participation|punch-card
gh-ai repo-traffic o/r | repo-community o/r | codeowners o/r
gh-ai merge-upstream o/r branch | merge-commit o/r base head
```

### Contents & git data
```bash
gh-ai repo-tree o/r [--max 200] | dir-list o/r [path]
gh-ai file-read o/r path [--start N] [--end N]
gh-ai file-write o/r path --content "..." [--file f] --message "..." [--branch b] [--sha ...]
gh-ai file-delete o/r path --message "..." | readme o/r
gh-ai git-refs o/r [--ref heads/main] | git-ref-create o/r REF SHA | git-ref-delete o/r REF --yes
gh-ai git-tree o/r SHA | git-blob o/r SHA
gh-ai status-create o/r SHA --state success --context ci | check-runs o/r SHA
```

### Commits & branches
```bash
gh-ai commit-list o/r [--branch b] [--path f] | commit-view o/r SHA | commit-compare o/r base...head
gh-ai commit-search "q" | branch-create o/r name [--from b] | branch-delete o/r name --yes
gh-ai branch-protect o/r branch [--require-reviews N] [--require-status] [--enforce-admins] [--off]
```

### Issues
```bash
gh-ai issue-list o/r [--state open|closed|all] [--label bug] [--assignee u] [--author u]
gh-ai issue-view o/r N | issue-create o/r --title "..." [--label bug]
gh-ai issue-update o/r N [--title x] [--state open|closed] [--add-label x] [--remove-label y]
gh-ai issue-close o/r N | issue-reopen o/r N | issue-labels | issue-milestones
gh-ai issue-milestone-create o/r "v1.0" | issue-events o/r N
```

### Pull requests
```bash
gh-ai pr-list o/r [--state open] [--author u] [--label bug] [--base main]
gh-ai pr-view o/r N | pr-create o/r --title "..." --head f --base main [--draft]
gh-ai create-pr o/r main branch --title "..." --body "..." [--dry-run]    # macro
gh-ai pr-update o/r N | pr-close o/r N | pr-merge o/r N [--squash|--rebase] [--delete-branch]
gh-ai pr-comment o/r N | pr-review o/r N --approve|--request-changes [--body "..."]
gh-ai pr-review-comment o/r N --path f --line 42 --body "..."
gh-ai pr-comments o/r N | pr-files o/r N | pr-reviewers o/r N [--add u]
gh-ai pr-checks o/r N | pr-diff o/r N | pr-status o/r N
```

### GitHub Actions
```bash
gh-ai action-list o/r | action-runs o/r [--branch b] [--status failure]
gh-ai action-run o/r RUN_ID | action-jobs o/r RUN_ID
gh-ai action-trigger o/r ci.yml --ref main [--inputs '{"k":"v"}']
gh-ai action-cancel o/r RUN_ID | action-rerun o/r RUN_ID [--failed]
gh-ai get-action-errors o/r RUN_ID                      # error blocks ±10 lines
gh-ai action-secret o/r [--set N --value v | --delete N] | action-variable o/r [--set|--delete]
gh-ai action-cache o/r | artifact-list o/r | artifact-dl o/r ID
gh-ai deploy-list o/r | deploy-create o/r ref [--environment prod] | deploy-status o/r DEP_ID [--state]
gh-ai pages o/r | pages-build o/r
```

### GitHub Agents (web: repo "Agents" tab)
```bash
gh-ai agent-tasks o/r | agent-task o/r TASK_ID
gh-ai agent-secret o/r [--set|--delete] | agent-variable o/r [--set|--delete]
```

### Security
```bash
gh-ai dependabot o/r [--state open] | code-scan o/r | secret-scan o/r
gh-ai vuln-alert o/r --enable|--disable
gh-ai advisory-list [o/r | --org ORG] | advisory-view o/r GHSA
```

### Projects v2 (GraphQL)
```bash
gh-ai project-list [--owner | --org ORG | --repo o/r]
gh-ai project-create "title" [--org ORG | --repo o/r]   # owner-scoped; --repo views fall back to owner
gh-ai project-view N [--org | --repo o/r]               # fields + items + values
gh-ai project-fields N [--org | --repo o/r]             # single-select options for --set-field
gh-ai project-add-item N --repo o/r [--issue N | --pr N]
gh-ai project-set-field N --item-id ID --field Status --value "In Progress"
gh-ai project-delete N --yes
```

### Discussions (GraphQL)
```bash
gh-ai discussion-list o/r | discussion-view o/r N | discussion-categories o/r
gh-ai discussion-create o/r --title "..." --category General --body "..."
gh-ai discussion-comment o/r N --body "..."
```

### Search
```bash
gh-ai search "q" --type repos|issues|code|users|commits|topics [--max 5]
gh-ai search-code "q" [--repo o/r]
```

### Activity & social
```bash
gh-ai star o/r | unstar o/r | starred [--login u] | repo-watchers o/r
gh-ai watch o/r [--ignore] | unwatch o/r | notif-sub o/r
gh-ai follow u | unfollow u | followers [--login] | following [--login]
gh-ai events [--repo o/r | --org ORG | --login u] [--received]
```

### Users / orgs / teams / gists / packages / wiki
```bash
gh-ai user-view [login] | user-repos | user-gists | user-keys | user-emails
gh-ai org-view ORG | org-members ORG | org-repos ORG | org-teams ORG
gh-ai org-invite ORG USER [--role] | org-outside ORG | org-blocks ORG [--block u]
gh-ai team-members ORG TEAM | team-repos ORG TEAM
gh-ai gist-list | gist-create --description "..." --file a.py [--stdin-name f] | gist-view ID
gh-ai gist-delete ID --yes | gist-star ID | gist-unstar ID | gist-fork ID
gh-ai package-list [--owner u | --org ORG | --repo o/r] [--type npm]
gh-ai package-view OWNER TYPE NAME | wiki o/r [--read PAGE]
```

### Repo administration
```bash
gh-ai repo-collab o/r [--add u [--permission push] | --remove u]   # list by default
gh-ai repo-hook o/r [--create URL [--events push,issues] | --delete ID | --ping ID]
gh-ai repo-key o/r [--add title --key "ssh-ed25519 ..." | --delete ID]
gh-ai ruleset-list [o/r | ORG --org] | ruleset-delete TARGET ID --yes
```

### Notifications & patches
```bash
gh-ai notif-list [--all] | notif-read [--thread ID]
cat diff.patch | gh-ai apply-patch file.py     # unified diff or search/replace
```

## Output Rules

- No URLs, node IDs, avatars, gravatars; reactions `+1:4, heart:6`; labels comma-separated
- File/log output capped 500 lines; lists default 20 (`--max` to change)
- Binary/media paths filtered from repo-tree; line numbers on file/diff output

## Error Codes

| Exit | Meaning |
|------|---------|
| 1 | Input error (no token, no stdin, missing confirm) |
| 2 | Network error after retry |
| 3 | 401 — bad credentials |
| 4 | 403 — rate limited or forbidden |
| 5 | 404 — not found |
| 6 | Other HTTP error / GraphQL error |
| 7-9 | Git failures |

## Known GitHub quirks (handled)

- `project-create --repo` makes an owner-scoped board (no GraphQL repo-link); `project-view/fields/delete --repo` fall back to the owner scope
- `/user/gists` can 404 despite gist scope; `gist-create` still works
- Code search empty for fresh repos (index lag); `repo-stats`/`codeowners` 404 until computed — reported gracefully
- Branch protection/rulesets need Pro on private repos (403)
- Dependabot/code-scan alerts 403 when feature disabled
- Destructive commands require `--yes`
