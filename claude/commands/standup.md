# /standup - Daily standup summary

Use Linear MCP tools to generate a standup summary.

## Steps

1. **Determine the lookback window:**
   - If today is **Monday**: use **3 days** (covers Friday through Sunday)
   - All other days: use **2 days**

2. **Fetch issues** using `mcp__claude_ai_Linear__list_teams` to find your team, then `mcp__claude_ai_Linear__list_issues` for your team:
   - Issues completed/done within the lookback window
   - Issues currently in progress (assigned to me)
   - Issues that are blocked

3. **Fetch recent GitHub activity:**
   - `git log --oneline --since="<N> days ago" --author="$(git config user.email)"` (use the lookback window from step 1)
   - `gh pr list --author @me --state all --limit 10 --json number,title,state,updatedAt` to find recent PRs

4. **Present the summary** in this format:

```
Yesterday:
✓ FIN-X: Issue title

Today (in progress):
→ FIN-X: Issue title

Blocked:
⊘ FIN-X: Issue title (reason if available)

Recent commits:
- repo: N commits
- PR #X merged/open: title
```

5. **Suggest `/next`** to pick up the next piece of work.

## Notes

- Skip sections that have nothing to show
- GitHub info requires `gh` CLI authenticated
