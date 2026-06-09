# /pr - Open a pull request for current work

Creates a PR from the current branch. Use this when you want review before the work is fully done, or when you want to separate "open PR" from "close issue".

For completed work, prefer `/done` — it handles commit, push, PR, and offers `/next` in one step.

## Usage

```
/pr                    # PR for current branch
/pr --draft            # Open as draft PR
/pr "custom title"     # Override the generated title
```

## Flow

1. **Detect the issue ID** from branch name (`TEAM-123` pattern)
   - If not found: ask "Which issue does this branch belong to? (e.g. FIN-42)"

2. **Show work summary:**
   ```bash
   git log --oneline <base>..HEAD
   git diff --stat <base>..HEAD
   ```

3. **Stage and commit** any uncommitted changes (ask what the commit message should be)

4. **Push:**
   ```bash
   git push -u origin HEAD
   ```

5. **Build PR title:** `"ISSUE-12: Issue title"` — use custom title if provided

6. **Build PR body:**
   - Brief bullet summary of what changed
   - `Closes ISSUE-12` at the end (required — triggers Linear integration)

7. **Create PR:**
   ```bash
   gh pr create --title "..." --body "..."
   # With --draft flag if requested
   ```

8. **Print the PR URL**

## If push fails due to diverged history

Follow the **github-cli** skill's "Push rejected (diverged history)" procedure. On conflict, tell the user: "Conflicts in `<files>`. Resolve, run `git rebase --continue`, then run `/pr` again." Never force-push.

## Notes

- Body **must** contain `Closes <ID>` — this triggers Linear's GitHub integration
- Base branch: detect with `git remote show origin | grep 'HEAD branch'`, default to `main`
- If a PR already exists for this branch (`gh pr view`), report the existing URL instead of creating a duplicate
- Do NOT run `linear done` — GitHub integration moves the Linear issue when the PR merges
- Follow the github-cli skill for PR body formatting
