# /split - Break a large issue into sub-issues

Decompose an L or XL issue into smaller, independently trackable sub-issues. Always shows the proposed breakdown before creating anything.

## Usage

```
/split ISSUE-12    # Split a specific issue
/split             # Split the current issue (detect from branch)
```

## Flow

1. **Load the issue:**
   ```bash
   linear issue show <id>
   ```

2. **Read the full description and acceptance criteria.** If the issue references code, read the relevant files.

3. **Draft a breakdown** using the **split skill** — follow it to find natural seams, identify sequencing, and size each sub-issue.

4. **Show the proposed breakdown — do not create yet:**
   ```
   Split FIN-12 into:
     1. FIN-?a: Design data model — S
        "Define the schema for X. Acceptance: schema reviewed and merged."
     2. FIN-?b: Implement API endpoints — M (blocked by 1)
        "Build POST/GET endpoints for X. Acceptance: endpoints tested, OpenAPI updated."
     3. FIN-?c: Build UI — M (blocked by 2)
        "Build the form and list view for X. Acceptance: matches design, responsive."
     4. FIN-?d: Write integration tests — S (blocked by 2)
        "E2E tests covering the create and list flows. Acceptance: CI green."

   Proceed?
   ```

5. **On confirmation, create sub-issues** — every sub-issue must include title, description, estimate, priority (inherit from parent unless there's reason to differ), and labels:
   ```bash
   linear issue create \
     --title "..." \
     --description "<one sentence what, one sentence acceptance criteria>" \
     --parent ISSUE-12 \
     --estimate S \
     --priority <parent priority>
   linear issue create \
     --title "..." \
     --description "..." \
     --parent ISSUE-12 \
     --estimate M \
     --priority <parent priority> \
     --blocked-by <prev>
   ```

6. **Update the parent** with a summary comment:
   ```bash
   linear issue comment ISSUE-12 "Split into sub-issues: ISSUE-13, ISSUE-14, ISSUE-15, ISSUE-16"
   ```
   Update parent estimate to XL if not already set.

7. **Offer to start the first unblocked sub-issue:** "Run `/next ISSUE-13` to begin."

## Notes

- Never auto-create — always show the proposed split first
- Keep sub-issues focused: one concern per issue
- Use `--blocked-by` to encode sequencing — this keeps `linear issues --unblocked` accurate
- The parent issue stays open until all sub-issues are done
- If a sub-issue is still M/L, that's fine — only split further if truly needed
