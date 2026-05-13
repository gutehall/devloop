# /sync - Sync Jira issues with current GitHub state

Detects and fixes drift between Jira issue status and the real state of branches and PRs.

## Usage

```
/sync             # Report drift (read-only, safe)
/sync fix         # Report drift and apply fixes (modifies Jira)
```

## Flow

### 1. Fetch in-progress issues

Run `jira issue list -a"$(jira me)" -s"In Progress" --plain` to get issues you currently have in progress.

### 2. Check each issue's branch and PR state

For each in-progress issue:

```bash
git branch --list "*<issue-key>*"                              # Does a local branch exist?
gh pr list --head <branch-name> --json number,state,mergedAt   # Any PR for that branch?
```

### 3. Classify each issue

| Situation | Classification |
|-----------|---------------|
| PR merged, issue still In Progress | Stale — should be Done |
| PR closed (not merged), issue In Progress | Abandoned — may need backlog |
| No branch or PR, issue In Progress | Stale — may need reset |
| Everything consistent | OK |

### 4. Report findings

Show a table:

```
Issue       Status (Jira)    PR State     Action needed
PROJ-42     In Progress      Merged       Move to Done
PROJ-38     In Progress      No PR        Check — no branch found
PROJ-31     In Progress      Open #47     OK
```

If everything is consistent, say so.

### 5. If `fix` flag is provided

For each issue needing action:

- **PR merged → Done**: run `jira issue move <id> "Done"` and confirm
- **Stale in-progress** (no branch/PR): ask before changing status
- **PR closed without merge**: ask before changing status

Never apply bulk fixes silently. Confirm each change or ask once before applying all.

### 6. Also check for unlinked PRs

```bash
gh pr list --state open --json number,title,headRefName
```

If a branch name matches the `[A-Z]+-[0-9]+` pattern but the corresponding Jira issue is not In Progress, flag it as a potential mismatch.

## Notes

- `/sync` (no flag) is always safe — read-only
- `/sync fix` modifies Jira state — be explicit about every change before applying
- Useful after returning from time off, after a sprint boundary, or when the board looks stale
- If `gh` is not authenticated or `jira` is not configured, say so and stop
