# /plan - Inline product planning with Jira

> **Project conventions:** (1) If this repo has a skill whose description says it defines coding rules, verify/test commands, or pre-merge gates — invoke it. (2) Else if `CLAUDE.md`, `AGENTS.md`, or `CONTRIBUTING.md` exists at the repo root (then the nearest subdirectory you are editing) — follow those. (3) Else continue.

Create Jira issues directly, without clipboard or intermediate steps.

This command warrants more reasoning than a routine turn, regardless of the session's default effort level — badly-scoped acceptance criteria cost everyone who implements or reviews the issue later. Take the extra time to think through edge cases and gaps before drafting.

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

4. **Create an epic and issues via CLI** directly:
   - First create an epic to group the work if one doesn't exist: `jira epic create -s"<epic name>" -b"<summary>"` — leave it in **Backlog** status (do not transition it forward)
   ```bash
   jira issue create -tStory -s"<title>" -b"<description>" --priority Medium
   ```
   - Print each created issue ID and title
   - Link to the parent epic: `jira issue create -tStory --parent PROJ-100 -s"..."`
   - New issues stay in **Backlog** — do **not** add them to the active sprint at planning time
   - Set at least one label on each issue (e.g. feature, bug, improvement)
   - Sequence the issues per the product-planning skill (priority + `blocks` links) so implementation order is visible, not just implied

5. **Offer next steps:**
   - Suggest `/estimate` to size the newly created issues before picking one up
   - Then suggest `/next <ID>` for the highest-priority issue once estimated
   - When implementation is done, use `/done` or `/pr` to push and open a PR

## Error Handling

- If `jira issue list` fails: say "Could not reach Jira. Is `jira` configured? Run `jira init` to set up." and stop.
- If no boards are found: ask the user to provide the board ID (run `jira board list` to check)
- If issue creation fails: report the specific error and do not silently skip. Offer to retry.
- If a duplicate issue is detected (same summary or very similar): show the existing issue and ask whether to proceed or update the existing one instead.

## Rules

- Always read current Jira state before planning — don't create duplicates
- Always group issues under an epic (check existing epics first) — new epics and issues start in **Backlog**, not the active sprint
- Issues need: summary, description, acceptance criteria, type, priority, and at least one label
- Issues must be sequenced (priority + `blocks` links) so the first issue to work on is clear
- Keep issues small enough to implement in one PR (Story/Task)
- Use Epic for large initiatives, Story for features, Bug for defects, Task for ops/chores
- No clipboard, no JSON payload, no external scripts
