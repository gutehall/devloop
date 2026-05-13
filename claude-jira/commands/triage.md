# /triage - Work through untriaged Jira issues interactively

Fetch all issues missing triage metadata and work through them one by one, suggesting values and applying confirmed changes.

## Usage

```
/triage              # All issues needing triage (no priority, no labels, no estimate)
/triage --priority   # Only issues with no priority set
/triage --unlabeled  # Only issues with no labels
```

## Flow

### 1. Load context

Run in parallel:
- `jira board list --plain` — find the active board
- `jira issue list --resolution Unresolved --plain` — all open issues
- `jira project list --plain` — available projects

Filter issues based on the flag passed:
- No flag: issues where priority is unset OR labels are empty OR story points/estimate is unset
- `--priority`: issues where priority is unset only
- `--unlabeled`: issues where labels are empty only

Report: `Found N issues needing triage.`

If 0 issues: say "All issues are triaged. Nothing to do." and stop.

If the CLI fails: say "Could not reach Jira. Check your configuration with `jira init`." and stop.

### 2. Work through issues one at a time

For each issue, run `jira issue view <key>`, then display:

```
--- Issue PROJ-42 ---
Title: <summary>
Type: <type>
Description: <first 200 chars, or "(no description)" if blank>
Current: priority=<unset|Highest|High|Medium|Low|Lowest>, labels=<none|label1, label2>
```

Analyze the summary and description to suggest values. Apply these rules:

**Priority suggestion:**
- Words like "crash", "broken", "data loss", "security", "blocked", "production" → High or Highest
- New features, improvements, refactors → Medium or Low
- Vague or very short descriptions → Low, note that more detail is needed
- Do not suggest a new priority if one is already set — leave it as-is

**Type suggestion:**
- Defect, broken, error, crash, wrong behavior → Bug
- New functionality, improvement, enhancement → Story
- Ops, chore, infrastructure, setup → Task
- Large multi-sprint initiative → Epic

**Labels suggestion:**
- Match on content: "bug" for defects, "security" for auth/vulnerability issues, "performance" for speed concerns
- Only suggest labels that make sense from the content — do not invent ones

**Epic suggestion:**
- Only suggest a parent epic if the issue clearly belongs to one based on its content
- If no obvious epic exists: leave it unset

Present suggestions as:

```
Suggested:
  priority: High
  type: Bug  (or "(keep current)" if already set)
  labels: [bug, auth]
  epic: PROJ-100 — Auth System  (or "(none)")

[Enter] Accept  [e] Edit  [s] Skip  [q] Quit
```

**On accept:** apply all suggested fields that are not already set:
```bash
jira issue edit <key> --priority High
jira issue edit <key> --label "bug,auth"
```
Print: `Updated PROJ-42.`

**On edit:** prompt for each field individually in sequence:
- `Priority [Highest / High / Medium / Low / Lowest / Enter to keep]:`
- `Type [Epic / Story / Bug / Task / Enter to keep]:`
- `Labels [comma-separated / Enter to keep]:`
- `Epic key [PROJ-100 / Enter to keep / n for none]:`

Then apply via `jira issue edit`. Print: `Updated PROJ-42.`

**On skip:** move to next issue. Print: `Skipped PROJ-42.`

**On quit:** stop immediately and go to summary.

### 3. Summary

```
Triaged N issues. Skipped M.
```

List each triaged issue key and summary on its own line.

## Error Handling

- If `jira issue edit` fails for an issue: print the error, offer to retry or skip, do not silently continue
- If the user quits mid-batch: report progress on issues already handled before stopping

## Rules

- Never override a field that already has a value, unless the user explicitly edits it
- Keep suggestions grounded in the issue content — do not guess randomly
- No clipboard, no JSON payloads, no external scripts
