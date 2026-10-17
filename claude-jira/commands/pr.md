# /pr - Open a pull request for current work

> **Project conventions:** (1) If this repo has a skill whose description says it defines coding rules, verify/test commands, or pre-merge gates — invoke it. (2) Else if `CLAUDE.md`, `AGENTS.md`, or `CONTRIBUTING.md` exists at the repo root (then the nearest subdirectory you are editing) — follow those. (3) Else continue.

Creates a PR from the current branch. Use this when you want review before the work is fully done, or when you want to separate "open PR" from "close issue".

For completed work, prefer `/done` — it handles commit, push, PR, and offers `/next` in one step.

## Usage

```
/pr                    # PR for current branch
/pr --draft            # Open as draft PR
/pr "custom title"     # Override the generated title
```

## Flow

1. **Detect the issue key** from branch name (`[A-Z]+-[0-9]+` pattern)
   - If not found: ask "Which issue does this branch belong to? (e.g. PROJ-42)"

2. **Detect `<base>` once** — `<base>` = origin HEAD branch (`git remote show origin | grep 'HEAD branch'`), else `main`; never `develop`. Reuse `git log --oneline <base>..HEAD` and `git diff --stat <base>..HEAD` later in this run.

3. **Show work summary** (reuse the log/stat from step 2)

4. **Quality gate** — follow the **ship-gate** skill in **interactive** mode (G1 commit, G2 verify, G3 size-gated review). `/pr --draft` / user wants a WIP PR: the gate allows pushing with `--draft` and stating failures in the body.

5. **Push:**
   ```bash
   git push -u origin HEAD
   ```

6. **Build PR title:** `"PROJ-12: Issue summary"` — use custom title if provided

7. **Build PR body:**
   - Brief bullet summary of what changed
   - `## Test plan` with the real commands and results from G2 — never an unchecked checkbox
   - `## Known issues` for unresolved Medium/Low findings from G3, if any
   - `Closes PROJ-12` at the end (triggers GitHub-Jira integration if configured)

8. **Create PR:**
   ```bash
   gh pr create --title "..." --body "..."
   # With --draft flag if requested
   ```

9. **Print the PR URL**

## If push fails due to diverged history

Follow the **github-cli** skill's "Push rejected (diverged history)" procedure. On conflict, tell the user: "Conflicts in `<files>`. Resolve, run `git rebase --continue`, then run `/pr` again." Never force-push.

## Notes

- Body **must** contain `Closes <KEY>` — triggers Jira-GitHub integration if installed
- If the integration is not set up, the issue will need to be manually moved: `jira issue move PROJ-12 "Done"` after merge
- If a PR already exists for this branch (`gh pr view`), report the existing URL instead of creating a duplicate
- Follow the github-cli skill for PR body formatting
