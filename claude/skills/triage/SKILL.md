---
name: triage
description: Interactively triage Linear issues missing priority, labels, or estimate. Only /triage, or when a command names this skill. Do not trigger proactively.
---

# Issue Triage

Work through untriaged Linear issues interactively, suggesting metadata based on issue content and applying confirmed values via MCP.

## Setup

Before triaging:

1. `mcp__claude_ai_Linear__list_teams` — identify active team (skip if already known from this session or `.linear` config)
2. `mcp__claude_ai_Linear__list_issue_labels` — fetch all available labels
3. `mcp__claude_ai_Linear__list_projects` — fetch all active projects
4. `mcp__claude_ai_Linear__list_issues` — fetch open issues, then filter to untriaged

An issue needs triage if any of the following are true:
- Priority is 0 (none)
- Labels list is empty
- Estimate is unset

## Large batches — delegate the fetch

For a batch of many untriaged issues (>~30), delegate the fetch-and-suggest loop (steps in "Suggesting values" below, for every issue) to a subagent and have it return a compact list of `issue: suggested priority/labels/estimate`. Then run the interaction loop below yourself against that list — the per-issue `get_issue` fetches don't need to sit in the main thread's context.

## Filtering

| Flag | Filter |
|------|--------|
| (none) | priority=none OR labels=empty OR estimate=unset |
| `--priority` | priority=none only |
| `--unlabeled` | labels=empty only |

## Suggesting values

For each issue, fetch full details via `mcp__claude_ai_Linear__get_issue` and analyze title and description.

### Priority

| Signal | Suggested priority |
|--------|--------------------|
| "crash", "broken", "data loss", "security", "blocked", "outage" | 1 Urgent or 2 High |
| Feature request, improvement, refactor | 3 Medium or 4 Low |
| Vague title, missing description | 4 Low — note more detail needed |
| Priority already set | Do not override |

### Labels

- Only suggest labels that exist in Linear — never invent new ones
- Match on content: bug reports → "bug", auth issues → "auth" (if label exists), etc.
- Suggest multiple labels if more than one applies
- Skip label suggestion if no labels exist in workspace

### Estimate

| Size | Criteria |
|------|----------|
| XS | Trivial change, one file, clearly understood |
| S | Small, well-scoped, a few files |
| M | Moderate scope, some uncertainty |
| L | Significant work, multiple components |
| XL | Large, poorly defined, or clearly multi-PR |
| (skip) | Estimate already set — do not override |

### Project

- Only suggest a project if the issue clearly belongs to one
- No obvious match → leave unset, don't force an assignment

## Interaction loop

Present one issue at a time:

```
--- Issue FIN-42 ---
Title: <title>
Description: <first 200 chars or "(no description)">
Current: priority=<value>, labels=<values or none>, estimate=<value or none>

Suggested:
  priority: High
  labels: [bug, auth]
  estimate: S
  project: Phase 1

[Enter] Accept  [e] Edit  [s] Skip  [q] Quit
```

**Accept:** apply all suggested values for fields not already set via `mcp__claude_ai_Linear__save_issue`.

**Edit:** prompt each field in sequence with current value shown; apply on confirmation.

**Skip:** move to next issue without changes.

**Quit:** stop and print the summary.

## Applying updates

Use `mcp__claude_ai_Linear__save_issue` with only the fields being changed. Never send a field already set unless user explicitly changed it during edit.

## Output

After all issues processed:

```
Triaged N issues. Skipped M.
```

Then one line per triaged issue: `ISSUE-ID: Title`

## Error handling

- `save_issue` failure: print the error, offer retry or skip — never silently continue
- Unknown label name during edit: warn and re-prompt
- MCP unreachable: say "Could not reach Linear. Check your MCP connection with `/mcp`." and stop

## Rules

- Never override a field that already has a value unless user explicitly edits it
- Only use labels that already exist in Linear
- Don't force project assignment
- Keep suggestions grounded in actual issue content

## Related skills

- **linear-cli** — full Linear CLI reference for bulk issue and label management
- **product-planning** — for planning new issues after a triage pass
