# /next - Find the next issue to work on

Works for any type of issue — code, documents, decks, reviews, planning, ops tasks.

## Usage

```
/next             # Show To Do issues to choose from
/next PROJ-12     # Skip selection, start working on PROJ-12 directly
```

## If no issue ID is provided

1. Run `jira issue list -s"To Do" -a"$(jira me)" --plain` to get your assigned issues ready to work on
2. If empty, run `jira issue list -s"To Do" --plain` for unassigned issues
3. Parse the output and count the issues
4. Present options using the format below

### Presenting Options

Present interactive options:
- Up to **3 issues** (own assigned issues first)
- **Always** include "Product planning" as an option
- If >3 issues, note how many more
- User can always type a specific issue ID

## Starting Work on an Issue

When the user selects an issue (or provides one directly via `/next PROJ-12`):

1. Run `jira issue move PROJ-12 "In Progress"` to set the status
2. Run `jira issue assign PROJ-12 "$(jira me)"` to assign to yourself
3. Run `jira issue view PROJ-12` to display full context
4. **Detect work type** (see below)
5. Follow the appropriate path

## Work Type Detection

Classify the issue based on its summary, description, type, and labels:

**Code work** — any of: implementation, bug fix, feature, refactor, migration, API, infrastructure, test. Also: working directory contains a git repo, or issue type is Bug/Story/Task.

**Non-code work** — any of: document, report, deck, presentation, plan, review, research, process, comms, meeting, analysis, spec, strategy.

If ambiguous, ask: "Is this code work or non-code work?"

---

## Path A: Code Work

1. Run `git pull` to sync the local repo before branching
2. Create a git branch from the issue:
   - Derive a slug from the issue summary (lowercase, spaces to dashes, strip special chars, max 50 chars)
   - Run: `git checkout -b PROJ-12-<slug>`
3. Read the full issue description and acceptance criteria from `jira issue view PROJ-12`
4. Explore relevant code (grep for symbols/files mentioned in the issue)
5. Implement — minimal solution, follow existing patterns, no over-engineering

### Implementation Rules

- Read relevant files before coding
- Minimal solution, focused changes only
- No unrelated refactors
- Follow existing patterns
- Check acceptance criteria before finishing
- If scope expands, stop and flag it
- Make the safest reasonable assumption on ambiguity; document it in a comment or PR

---

## Path B: Non-Code Work

1. Read the full issue description and acceptance criteria via `jira issue view PROJ-12`
2. Identify the deliverable (document, deck, plan, review, etc.)
3. Help produce it — draft, outline, structure, or content as appropriate
4. When output is ready, tell the user to run `/done` to close the issue

No git branch. No code. Focus entirely on producing the deliverable.

---

## If they choose "Product planning"

Ask what they want to focus on:
- Review and prioritize backlog
- Brainstorm new features
- Plan the next sprint

Then follow the product-planning skill guidelines.

## If no To Do issues are found

1. Run `jira issue list --resolution Unresolved --plain` to check what's actually there
2. **If all issues are In Progress or blocked:** show the list with status. Say: "No To Do issues. You can look at blocked issues or use `/plan` to add new work."
3. **If the backlog is empty:** say "No open issues found." and offer:
   - "Product planning" to create new issues with `/plan`
   - A prompt to describe something new: `/plan "..."`
4. **If `jira` is not configured:** if the CLI returns an auth error, say "Jira CLI is not set up. Run `jira init` to get started."

## Notes

- Issue IDs use project key prefix: `PROJ-123`
- `jira me` returns your Jira account ID — use it for self-assignment
- Jira statuses are workflow-specific; common values: "Backlog", "To Do", "In Progress", "In Review", "Done"
- Issue types: Epic, Story, Bug, Task, Sub-task
- The `--plain` flag suppresses interactive UI for script-friendly output
