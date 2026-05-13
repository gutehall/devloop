# /done - Complete work on an issue

Works for any type of issue — code, documents, decks, reviews, planning, ops tasks.

## Usage

```
/done             # Complete the current issue
/done PROJ-12     # Complete a specific issue
```

## Work Type Detection

Determine work type the same way `/next` does — issue summary, description, type, and whether a git branch matching the issue ID exists.

If a branch like `PROJ-42-*` is checked out, treat as code work.
If no such branch exists, treat as non-code work.
If ambiguous, ask.

---

## Path A: Code Work

1. **Detect the issue** from the branch name (extract `[A-Z]+-[0-9]+` pattern) or use the provided ID
2. **Show work summary:**
   - `git log --oneline <base>..HEAD`
   - `git diff --stat <base>..HEAD`
3. **Stage and commit** any uncommitted changes (if any exist)
4. **Push branch** to origin
5. **Create PR:**
   ```
   gh pr create --title "PROJ-12: Issue summary" --body "## Summary\n...\n\nCloses PROJ-12"
   ```
   Print the PR URL immediately after creation.
6. **Wait for CI:**
   ```bash
   gh pr checks --watch
   ```
   - All checks pass → proceed to merge
   - Any check fails → stop: say "CI failed. Fix the issue, push to the same branch, then run `/done` again." Do not merge.
   - No checks configured → proceed to merge immediately
7. **Merge:**
   ```bash
   gh pr merge --squash --delete-branch
   ```
8. **Return to base and pull:**
   ```bash
   git checkout <base>
   git pull
   ```
9. **Confirm merge landed:**
   ```bash
   git log --oneline -5
   ```
10. Offer `/next` to continue

### Branch name detection fallback

If the current branch does **not** match the `[A-Z]+-[0-9]+` pattern (e.g., it was created manually):
1. Run `git log --oneline -5` to show recent commits for context
2. Ask: "I couldn't detect an issue ID from branch `<branch-name>`. Which issue does this complete? (e.g., PROJ-42)"
3. Continue with the provided ID

### If push is rejected due to diverged history

If `git push` fails because the remote has diverged:
1. Run `git fetch origin` then `git log --oneline HEAD..origin/<branch>` to assess the gap
2. Run `git rebase origin/<base-branch>` (prefer rebase over merge for a clean history)
3. If conflicts appear: list the conflicting files and **stop** — do not auto-resolve
4. Tell the user: "There are conflicts in: `<files>`. Resolve them, then run `git rebase --continue` and `/done` again."
5. Do **not** force-push unless the user explicitly requests it

### Code Rules

- The PR title and body **must** contain the issue key (e.g., `PROJ-42`) — this links the PR to the Jira issue in the development panel
- Including `Closes PROJ-42` in the PR body auto-transitions the issue to Done on merge if the Jira-GitHub integration is configured
- If the Jira-GitHub integration is **not** set up: after merge, manually run `jira issue move PROJ-42 "Done"` to close the issue
- If there are no commits, skip the PR step and note it
- Base branch is typically `main` — detect with `git remote show origin | grep 'HEAD branch'` if unsure

---

## Path B: Non-Code Work

1. **Identify the issue** from the provided ID, or ask which issue was just completed
2. **Summarize** what was produced (document title, deck name, decision made, etc.)
3. **Ask for an artifact link:** "Any link to attach? (Google Doc, Slides, Confluence, Notion, etc.)"
   - If yes: run `jira issue comment add PROJ-42 "Artifact: <link>"` before closing
   - If no: skip
4. **Close in Jira:** run `jira issue move PROJ-42 "Done"`
5. Offer `/next` to continue

No git, no PR, no branch. Just capture the output and close.

---

## Notes

- The same command works regardless of role — manager closing a deck, engineer shipping a PR
- When in doubt about work type, check for a matching git branch first
- Smart Commits also work in commit messages (requires Jira DVCS connector): `git commit -m "PROJ-42 #done #comment Fixed the thing"`
