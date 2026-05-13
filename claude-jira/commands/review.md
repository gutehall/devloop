# /review - Review open pull requests

## Usage

```
/review           # List open PRs and select one to review
/review <PR#>     # Review a specific PR directly
```

## Flow

### 1. Fetch open PRs

```bash
gh pr list --state open --json number,title,author,headRefName,createdAt,isDraft
```

- Filter out draft PRs by default (include if user asks)
- Display as a numbered list: #, title, author, branch, age
- If no open PRs: say so and offer `/next` to continue working

### 2. User selects a PR

By number from the list, or via `/review <PR#>` directly.

### 3. Review the PR

Run in order:

```bash
gh pr view <number>        # Description, reviewers, CI summary
gh pr checks <number>      # CI check details
gh pr diff <number>        # Full diff
```

### 4. Review the diff

Follow the **code-review skill** to assess the PR. Cover: correctness, tests, security, and scope. For large diffs, summarize by file/area rather than line-by-line.

### 5. Summarize findings

Report:
- **What changed**: files touched, scope of change
- **CI status**: passing / failing (which checks)
- **Jira issue**: extract the issue key from the PR title/body (e.g., `PROJ-42`) — run `jira issue view PROJ-42` for full context
- **Issues spotted**: per the code-review skill
- **Questions**: anything unclear that the author should clarify

### 6. Offer actions

```
a) Approve              → gh pr review <number> --approve
b) Request changes      → gh pr review <number> --request-changes -b "reason"
c) Comment only         → gh pr review <number> --comment -b "comment"
d) Check out locally    → gh co <number>
e) Skip                 → continue
```

## Notes

- You cannot approve your own PR — GitHub will reject it. Skip the approve option when the PR author matches `git config user.email`
- If CI is failing, do not approve — note which checks failed and suggest fixes
- Follow the code-review skill for what to check and how to communicate findings
