# /start - Begin work on a Jira issue

Sets up your working context — assign, move to In Progress, create branch, show full issue — without immediately implementing. Use this when you already know what to work on and just need the context loaded.

## Usage

```
/start PROJ-12    # Start a specific issue
/start            # Pick from To Do issues
```

## Flow

1. **If no ID given:** run `jira issue list -s"To Do" --plain`, show up to 5 options, let user pick
2. **Assign and set In Progress:**
   ```bash
   jira issue move PROJ-12 "In Progress"
   jira issue assign PROJ-12 "$(jira me)"
   ```
3. **Detect work type** (same logic as `/next`): code vs. non-code based on summary, description, issue type
4. **For code work:** derive a slug from the summary (lowercase, spaces to dashes, strip special chars, max 50 chars) and create a branch:
   ```bash
   git checkout -b PROJ-12-<slug>
   ```
5. **Show full context:** `jira issue view PROJ-12`
6. **Confirm:** "Branch `PROJ-12-<slug>` checked out. Ready when you are."

## Notes

- This sets up context only — it does not start implementing. Start coding when you're ready.
- If the branch already exists locally, check it out without erroring
- For non-code issues, skip the branch step
- When done, run `/done` to create the PR and close the issue
- Prefer `/next` if you want to find work AND start implementing in one step
