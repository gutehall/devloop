---
name: retro
description: Run a sprint retrospective by pulling completed and in-progress work from Jira and git/GitHub, then create Jira issues for action items. Use this skill whenever the user runs /retro or asks for a sprint retrospective.
---

# Sprint Retrospective

Surface what shipped, what slipped, and what never started — then turn patterns into Jira issues.

## Setup

Before fetching data:

1. `jira board list --plain` — find the active board ID
2. `jira sprint list --board-id <id> --plain` — check for a recently completed sprint

## Time window resolution

Resolve the period in this order:

1. If a completed sprint exists with end date within the last 14 days: use that sprint's period
2. If the user passed `Nw`: start = N weeks ago, end = today
3. If the user passed `YYYY-MM-DD`: start = that date, end = today
4. Default: start = 14 days ago, end = today

Always print the resolved period before any data output.

## Data gathering

Fetch all sources in parallel.

### Jira issues

Use the resolved sprint name or time window:

| Category | Command |
|----------|---------|
| Shipped | `jira issue list -s"Done" --sprint "<sprint name>" --plain` |
| Slipped | `jira issue list -s"In Progress" --sprint "<sprint name>" --plain` |
| Never started | `jira issue list -s"To Do" --sprint "<sprint name>" --plain` |

For time-based windows (no sprint): filter `jira issue list --resolution Unresolved --plain` and check `updatedAt` against the start date.

### Git commits

```
git log --since=<start_date> --oneline --no-merges
```

Count total commits. Note the range for the velocity section.

### GitHub PRs

```
gh pr list --state merged --json number,title,mergedAt --limit 50
```

Filter client-side to mergedAt >= start_date.

```
gh pr list --state open --json number,title,createdAt
```

If `gh` fails or is not installed, continue with Jira data only and note the gap.

## Linking issues to PRs

For each shipped issue, check if a merged PR title or branch contains the issue key (e.g. `PROJ-42`). If matched, append `(PR #N)` to that line.

## Output format

```
## Retro: <start_date> → <end_date>

### Shipped
✓ PROJ-N: Issue summary (PR #N if available)

### Slipped (started but not finished)
→ PROJ-N: Issue summary — in sprint but not done

### Never started
○ PROJ-N: Issue summary — in sprint, status stayed To Do

### Velocity
- Closed: N issues
- PRs merged: N
- Commits: N
```

Omit sections with no items. Do not pad empty sections.

## Pattern analysis

After presenting the summary, ask the user whether to create action items. Do not create issues unprompted.

When the user confirms, scan the data for these patterns:

| Pattern | Signal | Action item type |
|---------|--------|-----------------|
| Recurring blockers | Same issue type blocked 2+ times | Root cause investigation issue |
| Chronic slippage | Same issue or area slips 2+ periods | Scoping or estimation issue |
| Empty sprints | Never-started count > shipped count | Planning or prioritization issue |
| No PR coverage | Commits with no associated PR | Process or workflow issue |
| Single reviewer | All PRs reviewed by same person | Team process issue |
| Late PR creation | PRs opened the same day as merge | Review process issue |

Only surface patterns actually present in the data. Do not invent findings.

## Creating action items

**Summary format:** `[Retro] Short description of the action`
Example: `[Retro] Resolve repeated auth-service blocking dependency`

**Description template:**
```
## Pattern observed
<what was seen in the retro data — be specific, reference issue keys or PR numbers>

## Proposed action
<what to do about it>

## Definition of done
<how you know this is resolved>
```

```bash
jira issue create -tTask \
  -s"[Retro] Action title" \
  -b"## Pattern observed\n...\n\n## Proposed action\n...\n\n## Definition of done\n..." \
  --priority Medium
```

Print each created issue key and summary after creation.

## Rules

- Never create action items without user confirmation
- One issue per pattern, not one per occurrence
- Keep titles action-oriented, not symptom-oriented
- Reference concrete data (issue keys, counts) in descriptions — do not generalize
- If the user dismisses a pattern, drop it without argument

## Related skills

- **product-planning** — for turning action items into structured work
- **jira-cli** — full CLI reference for issue management
