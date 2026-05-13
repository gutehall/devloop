# /scope - Audit a Jira project or epic for gaps

Review issues in a project or epic to find missing work, unclear requirements, or under-specified issues. Use this before a sprint to make sure nothing is falling through the cracks.

## Usage

```
/scope                    # Audit the current project
/scope "PROJ-100"         # Audit a specific epic
/scope --sprint "S12"     # Audit the current sprint
```

## Flow

1. **Load state:**
   ```bash
   jira board list --plain                              # Find board ID
   jira sprint list --board-id <id> --plain             # Active sprint context
   jira issue list --resolution Unresolved --plain      # All open issues
   jira epic list --plain                               # Epics
   # If scoping an epic:
   jira issue list --epics PROJ-100 --plain
   # If scoping a sprint:
   jira issue list --sprint "S12" --resolution Unresolved --plain
   ```
   Run `jira issue view` on each issue to get full descriptions.

2. **Read context:** Load `product.md` if it exists — understand intended scope before judging gaps.

3. **Analyze using the scope skill** — follow it for what to look for and how to reason about gaps.

4. **Present findings** grouped by category:
   ```
   Unclear (3): PROJ-4, PROJ-7, PROJ-12 — missing descriptions
   Unestimated (5): PROJ-8, PROJ-9, ...
   Not in sprint (2): PROJ-15, PROJ-16 — open but not assigned to any sprint
   Gaps: no issue covers user notification flow (mentioned in product.md §3)
   Oversized (1): PROJ-2 (XL) — should be broken down
   ```

5. **For each finding, offer an action** — do not auto-create or auto-edit:
   - Unclear → offer to add description inline: `jira issue edit <key> --body "..."`
   - Unestimated → offer `/estimate` for the batch
   - Not in sprint → offer to add: `jira sprint add --board-id <id> <key>`
   - Gap → ask for details before creating: `jira issue create -tStory -s"..." -b"..."`
   - Oversized → suggest `/split <key>`

## Notes

- Follow the scope skill for what to look for and how to present findings
- Read `product.md` before judging scope gaps — never speculate without product context
- Ask before creating any new issues — don't pad the backlog speculatively
- "Orphaned" in Jira terms = issues not in any active sprint and not part of any epic
