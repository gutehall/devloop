# /plan - Inline product planning with Linear

Create Linear issues directly via MCP, without clipboard or intermediate steps.

## Usage

```
/plan                    # Open-ended planning session
/plan "Add X feature"    # Focused planning for a specific area
```

## Flow

1. **Read Linear state:**
   - List open issues (`mcp__claude_ai_Linear__list_issues`)
   - List projects (`mcp__claude_ai_Linear__list_projects`)
   - List teams to get the correct team ID (`mcp__claude_ai_Linear__list_teams`)
   - List available labels (`mcp__claude_ai_Linear__list_issue_labels`)

2. **If input is vague or missing, ask three questions before doing anything else:**
   - What needs to happen?
   - Who is it for?
   - How do you know it's done?
   Draft the issue from those answers. Skip this step if the input already answers all three.

3. **Run product planning** inline — follow the product-planning skill guidelines:
   - Understand what's already planned
   - Identify gaps, priorities, or the focus area provided
   - Draft issues with clear titles, descriptions, and acceptance criteria

4. **Create a project and issues via MCP** directly:
   - First create a project with `mcp__claude_ai_Linear__save_project` if one doesn't exist for this work
   - Use `mcp__claude_ai_Linear__save_issue` for each issue — every issue **must** include:
     - `title` — clear, action-oriented
     - `description` — what, why, and acceptance criteria
     - `priority` — 1 (urgent) / 2 (high) / 3 (medium) / 4 (low)
     - `labelIds` — at least one label (e.g. feature, bug, improvement)
     - `teamId` and `projectId`
   - Print each created issue ID and title
   - After all issues are created, sort them by implementation order using `mcp__claude_ai_Linear__save_issue` (pass the existing issue `id` plus `sortOrder`) so issues appear in the sequence they should be worked on

5. **Offer next steps:**
   - Suggest `/estimate` to size the newly created issues before picking one up
   - Then suggest `/next <ID>` for the highest-priority issue once estimated
   - When implementation is done, use `/done` or `/pr` to push and open a PR

## Error Handling

- If `mcp__claude_ai_Linear__list_issues` or `list_projects` fails: say "Could not reach Linear. Is the MCP server connected? Run `/mcp` to check." and stop.
- If no teams are returned: say "No teams found in Linear. Make sure the MCP server is authenticated."
- If issue creation fails: report the specific error and do not silently skip. Offer to retry.
- If a duplicate issue is detected (same title or very similar): show the existing issue and ask whether to proceed or update the existing one instead.

## Rules

- Always read current Linear state before planning — don't create duplicates
- Always create a project to group the issues under (check existing projects first)
- Every issue **must** have: title, description, acceptance criteria, priority, and at least one label
- Issues must be sorted by implementation order — the first issue to work on appears first in Linear
- Keep issues small enough to implement in one PR
- No clipboard, no JSON payload, no external scripts
