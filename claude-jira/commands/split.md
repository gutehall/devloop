# /split - Break a large issue into child issues

Decompose an L or XL issue into smaller, independently trackable child issues. Always shows the proposed breakdown before creating anything.

## Usage

```
/split PROJ-12    # Split a specific issue
/split            # Split the current issue (detect from branch)
```

## Flow

1. **Load the issue:**
   ```bash
   jira issue view <key>
   ```

2. **Read the full description and acceptance criteria.** If the issue references code, read the relevant files.

3. **Draft a breakdown** using the **split skill** — follow it to find natural seams, identify sequencing, and size each child issue.

4. **Show the proposed breakdown — do not create yet:**
   ```
   Split PROJ-12 into:
     1. PROJ-?: Design data model — S
     2. PROJ-?: Implement API endpoints — M (blocked by 1)
     3. PROJ-?: Build UI — M (blocked by 2)
     4. PROJ-?: Write integration tests — S (blocked by 2)

   Proceed?
   ```

5. **On confirmation, create child issues:**
   ```bash
   jira issue create -tTask -s"Design data model" --parent PROJ-12
   # Record the new key, then link blocked issues:
   jira issue link PROJ-13 PROJ-12 "blocks"   # PROJ-13 must complete before PROJ-14
   jira issue create -tTask -s"Implement API endpoints" --parent PROJ-12
   jira issue link PROJ-14 PROJ-13 "blocks"
   ```

6. **Add a comment to the parent** naming the child issues:
   ```bash
   jira issue comment add PROJ-12 "Split into child issues: PROJ-13, PROJ-14, PROJ-15, PROJ-16"
   ```
   Update the parent priority to Highest if not already, to signal it's a large initiative.

7. **Offer to start the first unblocked child issue:** "Run `/start PROJ-13` to begin."

## Notes

- Never auto-create — always show the proposed split first
- Use `jira issue link <child> <dependency> "blocks"` to encode sequencing
- The `link` direction: the blocker "blocks" the blocked issue
- The parent issue stays open until all child issues are done: `jira issue move PROJ-12 "Done"`
- If a child is still M/L, that's fine — only split further if truly needed
