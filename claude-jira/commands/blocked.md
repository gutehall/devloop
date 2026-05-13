# /blocked - Mark an issue as blocked and document the blocker

## Usage

```
/blocked                     # Block the current issue (detect from branch)
/blocked PROJ-12             # Block a specific issue
/blocked PROJ-12 PROJ-99     # PROJ-12 is blocked by the existing PROJ-99
```

## Flow

### Detect the issue

If no ID is provided, detect from the current branch name (`[A-Z]+-[0-9]+` pattern).
If that fails, ask: "Which issue is blocked?"

---

### Case A: blocker is an existing issue

If the user provides a blocking issue ID:

1. Link the issues:
   ```bash
   jira issue link PROJ-12 PROJ-99 "blocks"
   ```
2. Add a comment to PROJ-12:
   ```bash
   jira issue comment add PROJ-12 "Blocked on PROJ-99"
   ```
3. Move PROJ-12 back to "To Do" or "Backlog" so it's clearly not active:
   ```bash
   jira issue move PROJ-12 "To Do"
   ```
4. Confirm: "PROJ-12 is now blocked by PROJ-99 and moved back to To Do."
5. Offer `/next` to find something else to work on

---

### Case B: blocker is unknown or external

If no blocker ID is provided:

1. Ask: "What's blocking this? Describe it or give an issue ID."
2. If they describe something external (waiting on credentials, a third party, a decision):
   ```bash
   jira issue create -tTask -s"<blocker description>" --priority High
   jira issue link PROJ-12 <NEW-KEY> "blocks"
   jira issue comment add PROJ-12 "Blocked on <NEW-KEY>: <description>"
   jira issue move PROJ-12 "To Do"
   ```
3. Show the created issue key
4. Confirm: "Created <NEW-KEY> as a blocker. PROJ-12 moved back to To Do."
5. Offer `/next` to continue with other work

---

## Notes

- Always offer `/next` after blocking — keep momentum going
- Use priority High for new blocker issues so they surface immediately
- Jira doesn't have a native "blocked" filter like Linear's `--unblocked`; moving the issue to "To Do" is the best equivalent to signal it's not actively in progress
- To unblock: resolve or close the blocker issue, then `jira issue move PROJ-12 "In Progress"`
