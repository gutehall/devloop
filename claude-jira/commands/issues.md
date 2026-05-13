# /issues - Browse and filter issues by sprint or epic

## Usage

```
/issues                  # Browse active sprint issues
/issues <sprint name>    # Jump directly to a named sprint
/issues epics            # Browse by epic instead
```

## Flow

### 1. Fetch context

Find the board ID: `jira board list --plain`

Display active boards as a numbered list. If only one board, use it automatically.

### 2. List current sprint

```bash
jira sprint list --board-id <id> --plain
```

Display active and upcoming sprints:

```
#   Name              Status       
1   Sprint 12         Active       
2   Sprint 13         Future       
```

If a sprint name was passed directly (e.g., `/issues Sprint 12`), skip the list.
If `/issues epics` was passed: skip to the epic view (see below).

### 3. User selects a sprint

By number from the list.

### 4. Fetch and display issues

```bash
jira issue list --sprint "Sprint 12" --resolution Unresolved --plain
```

Display as a table:

```
ID         Pri       Status          Type    Title                          Assignee
PROJ-42    High      In Progress     Story   Fix auth token expiry          @alice
PROJ-38    Medium    To Do           Task    Add retry logic                —
PROJ-31    Low       To Do           Bug     Improve error messages         —
```

If the sprint has no open issues: say "No open issues in <sprint>. Use `/plan` to add some."

### 4b. Epic view (if `/issues epics` was used)

```bash
jira epic list --plain
```

Show a numbered epic list. User selects one, then fetch child issues:

```bash
jira issue list --epics PROJ-100 --plain
```

### 5. Offer filter options

After showing the default list, offer:

```
Filter by status:   [1] To Do  [2] In Progress  [3] In Review  [4] All
Filter by assignee: [m] Mine  [u] Unassigned  [a] All
```

Apply filters and redisplay if selected.

### 6. Issue detail on demand

If the user types an issue ID (e.g., `PROJ-42`):
- Run `jira issue view PROJ-42`
- Display: summary, description, assignee, type, priority, labels, linked issues, comments
- Offer: `/next PROJ-42` to start work, or press Enter to return to the list

## Error Handling

- CLI error → "Could not reach Jira. Check your configuration with `jira init`."
- Sprint/epic not found by name → show the full list and ask to select

## Notes

- Always show issue IDs so the user can run `/next <ID>` directly
- Linked blocking issues are visible in `jira issue view` output
