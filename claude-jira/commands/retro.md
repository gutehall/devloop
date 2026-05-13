# /retro - Sprint retrospective

Pull completed and in-progress work from Jira and git/GitHub, then create action items as Jira issues.

## Usage

```
/retro              # Last completed sprint
/retro 1w           # Last week
/retro 3w           # Last 3 weeks
/retro 2024-01-01   # Since a specific date
```

## Steps

### 1. Determine the time window

- Run `jira sprint list --board-id <id> --plain` to find the most recently completed sprint
- If a completed sprint exists and its end date is within the last 2 weeks: use that sprint's period
- Otherwise: parse the argument to determine the lookback date:
  - No argument → 2 weeks ago
  - `Nw` → N weeks ago
  - `YYYY-MM-DD` → that exact date
- Print the resolved period: `Period: YYYY-MM-DD → YYYY-MM-DD`
- Find board ID first with `jira board list --plain` if not already known

### 2. Gather data (run in parallel)

**From Jira:**
- Issues completed in the period: `jira issue list -s"Done" --sprint "<sprint name>" --plain`
- Issues still In Progress that were in the sprint: `jira issue list -s"In Progress" --sprint "<sprint name>" --plain`
- Issues that slipped (carried over from the sprint with no progress): `jira issue list -s"To Do" --sprint "<sprint name>" --plain`

**From git/GitHub:**
- `git log --since=<start_date> --oneline --no-merges` — commits in period
- `gh pr list --state merged --json number,title,mergedAt --limit 50` — filter by mergedAt >= start_date
- `gh pr list --state open --json number,title,createdAt` — PRs still open

### 3. Present the retrospective

Use this exact format. Omit any section that has no items.

```
## Retro: <start_date> → <end_date>

### Shipped
✓ PROJ-N: Issue summary (PR #N if available)
...

### Slipped (started but not finished)
→ PROJ-N: Issue summary — in sprint but not done
...

### Never started
○ PROJ-N: Issue summary — in sprint, status stayed To Do
...

### Velocity
- Closed: N issues
- PRs merged: N
- Commits: N
```

### 4. Identify patterns and create action items

After presenting the summary, ask exactly:

> "Any patterns worth addressing? I can create Jira issues for action items."

If the user confirms or describes patterns, help identify concrete actions. Examples:
- Recurring blockers → issue to resolve the root cause
- Issues that keep slipping → scoping or estimation issue
- No tests in shipped code → tech-debt issue
- Single author on all PRs → process or pairing issue

Create each action item via:
```bash
jira issue create -tTask -s"<action title>" -b"<description>" --priority Medium
```

Print each created issue key and summary after creation.

## Error handling

- If `jira` returns no issues: say "No closed issues found in this period. Is `jira` configured? Run `jira init` to check." and stop.
- If `gh` is not available or returns an error: show the Jira summary only, note "GitHub data unavailable (gh not authenticated or not installed)."
- If no sprints exist: skip sprint detection silently and use the time-based window.
- If issue creation fails: report the specific error and offer to retry.

## Notes

- Keep action items small enough to implement in one PR
- Do not create action items unless the user asks for them
