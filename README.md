# gh-ai — AI-Native GitHub CLI

Token-optimized GitHub CLI designed for LLM agent consumption. Standard GitHub APIs return massive, deeply nested JSON — gh-ai strips all noise and combines multi-step operations into single macro commands.

Covers the **full GitHub surface** — REST + GraphQL + git protocol — with **172 commands**, mapped from a live crawl of the logged-in github.com web tree (Code · Issues · Pull requests · Agents · Actions · Projects · Wiki · Security · Insights · Settings, plus user/org/settings trees).

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

Requires Python 3, `click`, `requests`, `PyYAML` (+ `pynacl` only for Actions/Agents secret set). Set `GITHUB_TOKEN` (or `GITHUB_PERSONAL_ACCESS_TOKEN`, `GH_TOKEN`).

```bash
export GITHUB_TOKEN=$(gh auth token)   # if gh CLI is authenticated
```

### Token scopes

| Scope | Needed for |
| --- | --- |
| `repo` | Most read/write operations on repos |
| `workflow` | Writing/triggering Actions workflows, secrets |
| `delete_repo` | `repo-delete` |
| `gist` | `gist-*` write ops |
| `notifications` | `notif-list` / `notif-read` |
| `read:org` | Private org members/teams |
| `project` | GraphQL Projects v2 writes |

## Command Catalog (172)

### Info & utils
```
gh-ai rate-limit | auth-status | meta | zen | emojis | feeds
gh-ai license-list | gitignore-list T | gitignore-get T
gh-ai markdown "text"                     # Render markdown via API
gh-ai api /repos/o/r --method PATCH --data '{"description":"x"}'   # raw REST passthrough
gh-ai graphql 'query{viewer{login}}' [--var k=v]                     # raw GraphQL escape hatch
```

### Repositories
```
gh-ai repo-view o/r | repo-list [--owner u] | repo-init name [--private] [--org o] [--dry-run]
gh-ai repo-clone o/r [dir] | repo-fork o/r [--org o] | repo-delete o/r --yes
gh-ai repo-update o/r [--name x] [--description "..."] [--private|--public] [--default-branch main]
gh-ai repo-topics o/r [--set a --set b | --add a --remove b] | repo-langs o/r
gh-ai repo-contributors o/r | repo-branches o/r | repo-tags o/r | repo-license o/r
gh-ai repo-archive o/r --enable|--disable | repo-transfer o/r NEW_OWNER
gh-ai repo-generate TEMPLATE name [--owner u] [--private]        # from template repo
gh-ai repo-dl o/r [--ref b] [--zip] [--out f]                    # download archive
gh-ai repo-stats o/r --type contributors|commit-activity|code-frequency|participation|punch-card
gh-ai repo-traffic o/r | repo-community o/r | codeowners o/r
gh-ai merge-upstream o/r branch | merge-commit o/r base head
```

### Contents & git data
```
gh-ai repo-tree o/r [--max 200] | dir-list o/r [path] [--ref b]
gh-ai file-read o/r path [--start N] [--end N]
gh-ai file-write o/r path --content "..." [--file f] --message "..." [--branch b] [--sha ...]
gh-ai file-delete o/r path --message "..." [--branch b] | readme o/r [--ref b]
gh-ai git-refs o/r [--ref heads/main] | git-ref-create o/r REF SHA | git-ref-delete o/r REF --yes
gh-ai git-tree o/r SHA | git-blob o/r SHA
gh-ai status-create o/r SHA --state success --context ci [--description "..."]
gh-ai check-runs o/r SHA
```

### Commits & branches
```
gh-ai commit-list o/r [--branch b] [--path f] [--author u] | commit-view o/r SHA
gh-ai commit-compare o/r base...head | commit-search "q"
gh-ai branch-create o/r name [--from b] [--sha x] | branch-delete o/r name --yes
gh-ai branch-protect o/r branch [--require-reviews N] [--require-status] [--enforce-admins] [--off]
```

### Issues
```
gh-ai issue-list o/r [--state open|closed|all] [--label bug] [--assignee u] [--author u]
gh-ai issue-view o/r N | issue-create o/r --title "..." [--label bug] [--assignee u]
gh-ai issue-update o/r N [--title x] [--body x] [--state open|closed]
                       [--label a | --add-label x --remove-label y] [--assignee u] [--milestone 1]
gh-ai issue-close o/r N | issue-reopen o/r N
gh-ai issue-labels o/r | issue-milestones o/r | issue-milestone-create o/r "v1.0" | issue-events o/r N
```

### Pull requests
```
gh-ai pr-list o/r [--state open] [--author u] [--label bug] [--base main] [--max 20]
gh-ai pr-view o/r N | pr-create o/r --title "..." --head feature --base main [--draft]
gh-ai create-pr o/r main branch --title "..." --body "..." [--dry-run]   # macro: commit+push+PR
gh-ai pr-update o/r N [--title x] | pr-close o/r N
gh-ai pr-merge o/r N [--squash|--rebase] [--delete-branch] [--commit-title x]
gh-ai pr-comment o/r N [--body "..." | stdin] | pr-review o/r N --approve|--request-changes [--body "..."]
gh-ai pr-review-comment o/r N --path f --line 42 --body "..."
gh-ai pr-comments o/r N | pr-files o/r N | pr-reviewers o/r N [--add u] [--remove u]
gh-ai pr-checks o/r N | pr-diff o/r N [--max-lines 300] | pr-status o/r N
```

### GitHub Actions
```
gh-ai action-list o/r | action-runs o/r [--branch b] [--status failure] [--workflow ci.yml]
gh-ai action-run o/r RUN_ID | action-jobs o/r RUN_ID
gh-ai action-trigger o/r ci.yml --ref main [--inputs '{"k":"v"}']
gh-ai action-cancel o/r RUN_ID | action-rerun o/r RUN_ID [--failed]
gh-ai get-action-errors o/r RUN_ID                     # error blocks ±10 lines
gh-ai action-secret o/r [--set N --value v | --delete N] | action-variable o/r [--set|--delete]
gh-ai action-cache o/r | artifact-list o/r [--run-id] | artifact-dl o/r ID
gh-ai deploy-list o/r | deploy-create o/r ref [--environment prod] | deploy-status o/r DEP_ID [--state]
gh-ai pages o/r | pages-build o/r
```

### GitHub Agents (web: repo "Agents" tab)
```
gh-ai agent-tasks o/r [--max] | agent-task o/r TASK_ID
gh-ai agent-secret o/r [--set N --value v | --delete N] | agent-variable o/r [--set|--delete]
```

### Security
```
gh-ai dependabot o/r [--state open] [--severity high]    # Dependabot alerts
gh-ai code-scan o/r [--state open] [--severity high]     # Code scanning alerts
gh-ai secret-scan o/r [--state open]                     # Secret scanning alerts
gh-ai vuln-alert o/r --enable|--disable
gh-ai advisory-list [o/r | --org ORG] | advisory-view o/r GHSA-ID
```

### Projects v2 (GraphQL)
```
gh-ai project-list [--owner | --org ORG | --repo o/r]
gh-ai project-create "title" [--org ORG | --repo o/r]    # note: owner-scoped boards
gh-ai project-view N [--org | --repo o/r]                # fields + items + values
gh-ai project-fields N [--org | --repo o/r]              # incl. single-select options
gh-ai project-add-item N --repo o/r [--issue N | --pr N | --content-id ID]
gh-ai project-set-field N --item-id ID --field Status --value "In Progress"
gh-ai project-delete N --yes
```

### Discussions (GraphQL)
```
gh-ai discussion-list o/r | discussion-view o/r N | discussion-categories o/r
gh-ai discussion-create o/r --title "..." --category General --body "..."
gh-ai discussion-comment o/r N --body "..."
```

### Search
```
gh-ai search "q" --type repos|issues|code|users|commits|topics [--max 5]
gh-ai search-code "q" [--repo o/r]
```

### Activity & social
```
gh-ai star o/r | unstar o/r | starred [--login u] | repo-watchers o/r
gh-ai watch o/r [--ignore] | unwatch o/r | notif-sub o/r
gh-ai follow u | unfollow u | followers [--login] | following [--login]
gh-ai events [--repo o/r | --org ORG | --login u] [--received] [--max]
```

### Users, orgs, teams, gists, packages, wiki
```
gh-ai user-view [login] | user-repos [login] | user-gists [login] | user-keys [--login] | user-emails
gh-ai org-view ORG | org-members ORG | org-repos ORG | org-teams ORG
gh-ai org-invite ORG USER [--role direct_member] | org-outside ORG | org-blocks ORG [--block u]
gh-ai team-members ORG TEAM | team-repos ORG TEAM
gh-ai gist-list | gist-create --description "..." --file a.py [--stdin-name f] | gist-view ID
gh-ai gist-delete ID --yes | gist-star ID | gist-unstar ID | gist-fork ID
gh-ai package-list [--owner u | --org ORG | --repo o/r] [--type npm]   # all types by default
gh-ai package-view OWNER TYPE NAME
gh-ai wiki o/r [--read PAGE]                     # list/read wiki via .wiki.git clone
```

### Repo administration
```
gh-ai repo-collab o/r [--add u [--permission push] | --remove u]     # list by default
gh-ai repo-hook o/r [--create URL [--events push,issues] [--secret s] | --delete ID | --ping ID]
gh-ai repo-key o/r [--add title --key "ssh-ed25519 ..." | --delete ID]   # deploy keys
gh-ai ruleset-list [o/r | ORG --org] | ruleset-delete TARGET ID --yes
```

### Notifications
```
gh-ai notif-list [--all] [--participating] | notif-read [--thread ID]
```

### Patch application
```
cat diff.patch | gh-ai apply-patch file.py     # unified diff or search/replace block
```

## Web-tree Coverage Map

Live-crawled (logged in as lesterppo, 77 pages) and mapped to commands:

| github.com area | gh-ai commands |
| --- | --- |
| Code / files / git | repo-tree, dir-list, file-read/write/delete, readme, git-refs/blob/tree, repo-dl |
| Commits / branches / tags | commit-list/view/compare/search, branch-*, git-ref-*, repo-tags, branch-protect |
| Issues | issue-* (11 commands), issue-events |
| Pull requests | pr-* (17 commands), create-pr macro |
| Agents (new tab) | agent-tasks/task/secret/variable |
| Actions | action-* (10), get-action-errors, artifact-*, deploy-*, pages |
| Projects v2 | project-* (8, GraphQL) |
| Wiki | wiki (clone-based) |
| Security | dependabot, code-scan, secret-scan, vuln-alert, advisory-* |
| Insights | repo-stats, repo-traffic, repo-contributors, repo-community |
| User profile | user-view/repos/gists/keys/emails, starred, followers/following |
| Settings | repo-collab, repo-hook, repo-key, ruleset-*, action-secret/variable, branch-protect |
| Notifications | notif-list/read/sub |
| Search | search (6 types), search-code |
| GraphQL-only | project-*, discussion-*, graphql hatch |

## Token Efficiency Rules

- No `url`, `html_url`, `node_id`, `avatar_url`, `gravatar_id`
- Reactions: `+1:4, heart:6` (omitted if zero); labels as comma-separated string
- Max 500 lines per output; list commands default to 20 items (`--max` to change)
- Binary/media paths filtered from repo-tree; line numbers on all file/diff output

## Error Codes

| Exit | Meaning |
|------|---------|
| 1 | Input error (no token, no stdin for patch, missing confirm) |
| 2 | Network error after retry |
| 3 | 401 — bad credentials |
| 4 | 403 — rate limited or forbidden |
| 5 | 404 — not found |
| 6 | Other HTTP error (422 validation, GraphQL error) |
| 7 | Git command failed (clone/wiki) |
| 8 | Git command timed out |
| 9 | Git not found on PATH |

## Notes & known GitHub quirks

- `project-create --repo` creates an **owner-scoped** board (GraphQL has no repo-link mutation); `project-view/fields/delete --repo` fall back to the owner scope automatically
- `/user/gists` can 404 for some tokens even with gist scope; `gist-create` still works
- Code search may return empty for freshly pushed repos (`incomplete_results` index lag)
- `repo-stats` and `codeowners` may 404 until GitHub computes/indexes — reported gracefully
- Branch protection & rulesets require Pro on private repos (403 with clear message)
- Dependabot/code-scanning alerts 403 when the feature is disabled for the repo
- Destructive commands (`repo-delete`, `branch-delete`, `gist-delete`, `release-delete`, `ruleset-delete`, `project-delete`, `git-ref-delete`) require `--yes`

## Skills

- `skills/claude/SKILL.md` — Claude Code skill
- `skills/hermes/SKILL.md` — Hermes Agent skill

## License

MIT
