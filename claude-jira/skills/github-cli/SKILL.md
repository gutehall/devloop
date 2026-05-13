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
gh pr create --title "PROJ-12: Add feature" --body "..."
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
  --title "PROJ-12: Short description" \
  --body "$(cat <<'EOF'
## Summary

- What changed and why (bullet points)

## Test plan

- [ ] Tested locally
- [ ] Edge cases considered

Closes PROJ-12
EOF
)"
```

### Rules

- **Title format:** `PROJ-KEY: Short description` — links the PR to the Jira issue in the development panel
- **Body must end with `Closes PROJ-KEY`** — triggers Jira-GitHub integration to auto-transition the issue on merge (if configured)
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
gh pr merge --squash  # Squash commits into one clean commit on main
```

### View a specific PR
```bash
gh pr view 123
gh pr diff 123
```

## Related Skills

- **jira-cli** — for Jira issue management
- **product-planning** — for planning work before it becomes PRs
