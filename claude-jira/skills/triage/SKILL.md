---
name: triage
description: Interactively triage Jira issues that are missing priority, labels, estimate, or type. Use this skill whenever the user runs /triage or asks to triage, review, or clean up untriaged issues in Jira.
---

# Issue Triage

Work through untriaged Jira issues interactively, suggesting metadata based on issue content and applying confirmed values via CLI.

## Setup

Before triaging:

1. `jira board list --plain` — find the active board ID
2. `jira issue list --resolution Unresolved --plain` — fetch all open issues
3. `jira project list --plain` — fetch available projects

An issue needs triage if any of the following are true:
- Priority is unset (no priority)
- Labels list is empty
- Story points / estimate is unset

## Filtering

| Flag | Filter |
|------|--------|
| (none) | priority=unset OR labels=empty OR estimate=unset |
| `--priority` | priority=unset only |
| `--unlabeled` | labels=empty only |

## Suggesting values

For each issue, run `jira issue view <key>` and analyze the summary and description.

### Priority

| Signal | Suggested priority |
|--------|-------------------|
| "crash", "broken", "data loss", "security", "blocked", "production", "outage" | Highest or High |
| Feature request, improvement, refactor | Medium or Low |
| Vague title, missing description | Low — note that more detail is needed |
| Priority already set | Do not override |

### Type

| Signal | Suggested type |
|--------|---------------|
| Defect, broken, error, crash, wrong behavior | Bug |
| New functionality, improvement, enhancement | Story |
| Ops, chore, infrastructure, setup | Task |
| Large multi-sprint initiative | Epic |

### Labels

- Only suggest labels that make sense from the issue content
- Match on content: "bug" for defects, "security" for auth/vulnerability issues, "performance" for speed concerns
- Do not invent labels — suggest only common, logical values

### Epic

- Only suggest a parent epic if the issue clearly belongs to one
- If no obvious epic: leave it unset

## Interaction loop

Present one issue at a time:

```
--- Issue PROJ-42 ---
Title: <summary>
Type: <type>
Description: <first 200 chars or "(no description)">
Current: priority=<value or unset>, labels=<values or none>

Suggested:
  priority: High
  type: Bug  (or "(keep current)" if already set)
  labels: [bug, auth]
  epic: PROJ-100 — Auth System  (or "(none)")

[Enter] Accept  [e] Edit  [s] Skip  [q] Quit
```

**Accept:** apply all suggested fields that are not already set:
```bash
jira issue edit PROJ-42 --priority High
jira issue edit PROJ-42 --label "bug,auth"
```
Print: `Updated PROJ-42.`

**Edit:** prompt each field individually:
- `Priority [Highest / High / Medium / Low / Lowest / Enter to keep]:`
- `Type [Epic / Story / Bug / Task / Enter to keep]:`
- `Labels [comma-separated / Enter to keep]:`
- `Epic key [PROJ-100 / Enter to keep / n for none]:`

Then apply via `jira issue edit`. Print: `Updated PROJ-42.`

**Skip:** move to next issue. Print: `Skipped PROJ-42.`

**Quit:** stop and print the summary.

## Output

After all issues are processed:

```
Triaged N issues. Skipped M.
```

List each triaged issue key and summary.

## Error handling

- `jira issue edit` failure: print the error, offer retry or skip — do not silently continue
- If CLI fails: "Could not reach Jira. Check your configuration with `jira init`." and stop

## Rules

- Never override a field that already has a value unless the user explicitly edits it
- Keep suggestions grounded in actual issue content
- Do not force project or epic assignment

## Related skills

- **jira-cli** — full CLI reference for issue management
- **product-planning** — for planning new issues after a triage pass
