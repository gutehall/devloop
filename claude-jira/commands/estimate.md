# /estimate - Estimate unestimated Jira issues

Bulk-review issues missing story point estimates and assign sizes interactively.

## Usage

```
/estimate                    # Work through all unestimated issues
/estimate PROJ-12            # Estimate a specific issue
/estimate --sprint "S12"     # Estimate issues in a specific sprint
```

## Flow

1. **Fetch issues to estimate:**
   ```bash
   jira issue list --resolution Unresolved --plain
   # If --sprint given:
   jira issue list --sprint "S12" --resolution Unresolved --plain
   ```
   Filter to those with no story point estimate set.

2. **For each issue, one at a time:**
   - Run `jira issue view <key>` to get the full description
   - Read referenced code files if the description names them
   - Apply the **estimate skill** to reason about size
   - Suggest a t-shirt size with a one-line rationale:
     > "I'd say **M** (3 pts) — data model change plus a UI update, probably a day's work."
   - Ask: "M, or different?" — accept the suggestion, adjust, or skip

3. **On confirmation**, add the estimate as a comment (Jira story points depend on field config — use the method that works for your setup):
   ```bash
   # If story points field is available via CLI:
   jira issue edit <key> --custom "story_points=<n>"
   # Otherwise, record via comment:
   jira issue comment add <key> "Estimate: M (3 story points)"
   ```

4. **Continue** to the next unestimated issue

5. **At the end, summarize:**
   > "Estimated 6 issues. Largest: PROJ-12 (XL). 2 skipped."
   - For any XL issues, suggest: "PROJ-12 is XL — consider running `/split PROJ-12`"

## Size → story point mapping

| Size | Story Points | Meaning |
|------|-------------|---------|
| XS | 1 | Trivial, under an hour |
| S | 2 | A couple hours |
| M | 3 | About a day |
| L | 5 | Multi-day — consider breaking down |
| XL | 8 | Should definitely be broken down |

## Notes

- Follow the estimate skill for how to reason about size and what signals to use
- Skip issues you can't reason about: missing description, no acceptance criteria — note them at the end
- If L or XL, suggest `/split` but don't block the session on it
- Story points in Jira CLI may require custom field configuration; if `--custom` doesn't work, fall back to comments
