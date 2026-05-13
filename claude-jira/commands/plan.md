# /plan - Inline product planning with Jira

Create Jira issues directly, without clipboard or intermediate steps.

## Usage

```
/plan                    # Open-ended planning session
/plan "Add X feature"    # Focused planning for a specific area
```

## Flow

1. **Read Jira state:**
   - List open issues: `jira issue list --resolution Unresolved --plain`
   - List epics: `jira epic list --plain`
   - Find the active board and sprint: `jira board list --plain` then `jira sprint list --board-id <id> --plain`

2. **If input is vague or missing, ask three questions before doing anything else:**
   - What needs to happen?
   - Who is it for?
   - How do you know it's done?
   Draft the issue from those answers. Skip this step if the input already answers all three.

3. **Run product planning** inline — follow the product-planning skill guidelines:
   - Understand what's already planned
   - Identify gaps, priorities, or the focus area provided
   - Draft issues with clear summaries, descriptions, and acceptance criteria

4. **Create issues via CLI** directly:
   ```bash
   jira issue create -tStory -s"<title>" -b"<description>" --priority Medium
   ```
   - Print each created issue ID and title
   - Link to a parent epic if applicable: `jira issue create -tStory --parent PROJ-100 -s"..."`
   - Add to active sprint if appropriate: `jira sprint add --board-id <id> PROJ-123`

5. **Offer to start:**
   - Suggest `/next <ID>` for the highest-priority issue created
   - When implementation is done, use `/done` or `/pr` to push and open a PR

## Error Handling

- If `jira issue list` fails: say "Could not reach Jira. Is `jira` configured? Run `jira init` to set up." and stop.
- If no boards are found: ask the user to provide the board ID (run `jira board list` to check)
- If issue creation fails: report the specific error and do not silently skip. Offer to retry.
- If a duplicate issue is detected (same summary or very similar): show the existing issue and ask whether to proceed or update the existing one instead.

## Rules

- Always read current Jira state before planning — don't create duplicates
- Issues need: summary, description, acceptance criteria, type, priority
- Keep issues small enough to implement in one PR (Story/Task)
- Use Epic for large initiatives, Story for features, Bug for defects, Task for ops/chores
- No clipboard, no JSON payload, no external scripts
