---
name: github-cli
description: GitHub CLI (gh) reference for managing PRs, issues, and CI from the terminal. Used by /pr, /done, /review, and /sync commands.
allowed-tools: Bash(gh:*)
---

# GitHub CLI

Reference for `gh` — the official GitHub CLI. Used across this workflow for PR creation, review, and CI checks.

Install: `brew install gh` (or check https://cli.github.com)
Auth: `gh auth login`

## Quick Reference

```bash
# PRs
gh pr create --title "ISSUE-12: Add feature" --body "..."
gh pr list                          # Open PRs in this repo
gh pr view                          # Current branch's PR
gh pr view --web                    # Open in browser
gh pr status                        # Your PRs across repos
gh pr diff                          # Diff for current branch's PR
gh pr merge                         # Merge current PR (interactive)
gh pr merge --squash                # Squash and merge
gh pr merge --rebase                # Rebase and merge
gh pr review --approve              # Approve
gh pr review --request-changes -b "Needs work on X"
gh pr comment 123 --body "..."      # Comment on a PR by number
gh pr checks                        # CI status for current PR
gh pr close 123                     # Close without merging
gh pr reopen 123

# Issues
gh issue list                       # Open issues
gh issue view 123
gh issue create --title "..." --body "..."
gh issue close 123

# Repos
gh repo view                        # Current repo details
gh repo clone owner/repo

# CI Runs
gh run list                         # Recent workflow runs
gh run view                         # Current branch's run
gh run watch                        # Stream live run output
gh run rerun                        # Re-run failed run
```

## PR Creation Pattern

Always use this structure when creating PRs in this workflow:

```bash
gh pr create \
  --title "ISSUE-12: Short description" \
  --body "$(cat <<'EOF'
## Summary

- What changed and why (bullet points)

## Test plan

- `npm test` — 42 passed, 0 failed
- `npm run build` — clean

Closes ISSUE-12
EOF
)"
```

### Rules

- **Title format:** `ISSUE-ID: Short description` — enables Linear's GitHub integration to link the PR
- **Body must end with `Closes ISSUE-ID`** — triggers Linear to auto-close the issue when the PR merges
- **Test plan must contain the commands actually run and their actual results** — never unchecked checkboxes. If the repo has no test/build tooling, say so in the test plan.
- Do not use `--fill` — it produces poor titles and bodies
- Use `--draft` for WIP PRs not ready for review

## Detecting the base branch

```bash
git remote show origin | grep 'HEAD branch'   # Most reliable
git symbolic-ref refs/remotes/origin/HEAD      # Faster fallback
```

Default to `main` if detection fails.

## Check if a PR already exists

```bash
gh pr view --json number,title,url,state
```

Run this before `gh pr create` to avoid duplicate PRs.

## Common patterns

### Check CI before merging
```bash
gh pr checks          # Pass/fail summary for current PR
gh run watch          # Stream live output
```

### Handle a review
```bash
gh pr view            # See inline comments and review state
gh pr review --approve
gh pr review --request-changes -b "Please add tests for the edge case in auth.ts"
```

### Merge after CI passes
```bash
gh pr merge --squash --delete-branch          # interactive flows, after the user confirms
gh pr merge --auto --squash --delete-branch   # arm auto-merge: GitHub merges once checks + required reviews pass (use in /grind and /autopilot)
```

Never merge with failing checks. If the repo has no checks at all, treat that as a missing gate — flag it and set up branch protection (below) rather than merging unvalidated code.

### View a specific PR
```bash
gh pr view 123
gh pr diff 123
```

## Branch protection (server-side enforcement)

The workflow's quality gates (diff review, local tests, code-review skill) live in prompts — they are advisory. Branch protection on the default branch makes them unbypassable for every contributor, human or bot:

```bash
# Require CI checks + 1 approving review on main
gh api -X PUT repos/{owner}/{repo}/branches/main/protection --input - <<'EOF'
{
  "required_status_checks": { "strict": true, "contexts": ["<your-ci-check-name>"] },
  "required_pull_request_reviews": { "required_approving_review_count": 1 },
  "enforce_admins": true,
  "restrictions": null
}
EOF

# Allow PRs to be armed for auto-merge (used by /grind and /autopilot)
gh repo edit --enable-auto-merge
```

With protection in place:

- A direct `gh pr merge` fails until required checks and reviews are satisfied
- `gh pr merge --auto --squash --delete-branch` arms the merge; GitHub completes it once requirements are met
- `enforce_admins: true` closes the "admins can bypass" hole

Find the check name with `gh pr checks` on any open PR, or from `.github/workflows/*.yml` job names.

## Push rejected (diverged history)

Canonical procedure when `git push` is rejected because the remote branch moved on. All commands reference this — do not re-spell it.

1. `git fetch origin`, then check the gap: `git log --oneline HEAD..origin/<branch>`
2. `git rebase origin/<base>` — rebase, **never** merge
3. **Conflicts → stop. List the conflicting files. Do not auto-resolve.** The caller decides the outcome:
   - interactive command (`/pr`, `/done`) → tell the user to resolve, run `git rebase --continue`, then re-run the command
   - autonomous loop (`/grind`) → **STOP LOOP**
4. **Never force-push** unless the user explicitly requests it.

## Related Skills

- **linear-cli** — for Linear issue management
- **product-planning** — for planning work before it becomes PRs
