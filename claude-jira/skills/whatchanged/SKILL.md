---
name: whatchanged
description: Generate a management-facing progress report covering everything shipped since the last time the command was run — features delivered, bugs fixed, and work in flight. Persists a checkpoint so each run only covers new changes. Use this skill whenever the user runs /whatchanged or asks for a status report for management.
---

# What Changed — Management Report

Produce a plain-language progress report for a non-technical audience. Lead with shipped value, not implementation detail. Persist a checkpoint so each run covers only the period since the last report.

## State file

The checkpoint lives at `.claude/whatchanged` in the project root (create `.claude/` if it does not exist).

File format (plain text, two lines):

```
timestamp=<ISO-8601 datetime>
commit=<full SHA>
```

Example:

```
timestamp=2026-05-01T14:32:00Z
commit=a3f9d12e8b1c4f7a0e2d5b6c9f3a8e1d4b7c0f2
```

## Step 1: Read the checkpoint

```
cat .claude/whatchanged 2>/dev/null
```

- If the file does **not** exist: this is the first run. Print:

  > "No previous checkpoint found. Recording current state as the baseline — run `/whatchanged` again after your next batch of work to generate the first report."

  Then jump straight to **Step 5** (write the checkpoint and stop).

- If the file exists: parse `timestamp` and `commit` from it.

## Step 2: Gather data (run all in parallel)

Use `<timestamp>` as the time baseline for all sources and `<commit>` as the git baseline.

### Jira issues

```
jira issue list --updated-after <timestamp> --plain
```

Categorise each issue:
- **Shipped**: status moved to Done/Closed within the period
- **In progress**: status is In Progress and was not Done at the start of the period
- **Started**: moved from To Do/Backlog to In Progress within the period

For each shipped issue, note the issue type if available (Story, Bug, Task, Improvement) — this drives the report grouping in Step 3.

If `jira` is unavailable: note "Jira data unavailable" and continue with GitHub data only.

### Merged PRs

```
gh pr list --state merged --json number,title,mergedAt,author,body --limit 50
```

Filter client-side to `mergedAt` > `<timestamp>`.

For each merged PR, check whether its title or body contains a Jira issue ID (e.g. `PROJ-42`, `Closes PROJ-42`). PRs that reference a Jira issue are already covered by the Jira section — do not double-count them. PRs with **no Jira issue reference** are untracked work done directly in Claude Code or outside the issue tracker.

If `gh` is unavailable: skip silently and note at the bottom.

### Untracked commits

```
git log <commit>..HEAD --oneline --no-merges
```

Identify commits that are not covered by any merged PR in the list above. These are direct commits with no PR and no Jira issue.

If `<commit>` is no longer in history (e.g. after a rebase): fall back to `--since=<timestamp>`.

### Commit count (context only — not shown in body of report)

```
git log <commit>..HEAD --oneline --no-merges | wc -l
```

Used only in the footer line, not in the body.

## Step 3: Write the report

Write in plain language. Avoid technical jargon (no branch names, commit SHAs, file paths, or PR numbers in the body). Use issue summaries as written in Jira — they are already the business description.

Use this format. Omit any section that has no items.

```
## Progress Report: <start_date> → <end_date>

### Delivered
- <Issue summary or synthesized description> — <one sentence plain-language description of the value delivered>
- ...

### Bug Fixes
- <Issue summary or synthesized description>
- ...

### Other Work
- <synthesized description of untracked PR or commit>
- ...

### In Progress
- <Issue summary> — started <N> days ago
- ...

### Coming Up (started this period)
- <Issue summary>
- ...

---
<N> commits · <N> PRs merged · Period: <start_date> → <end_date>
```

**Writing the descriptions under Delivered:**
- For Jira issues: pull the description from the issue if it has one, otherwise infer from the summary
- For untracked PRs and commits: read the PR title or commit messages and synthesize a plain-language summary of what changed
- One sentence maximum — what the user/customer can now do, or what was improved, not what the engineer changed
- Example: "Users can now reset their password via email" not "Added POST /auth/reset endpoint"

**Grouping untracked work:**
- Place untracked PRs and direct commits under **Other Work** if they are unclear in nature
- If the PR title or commit messages clearly indicate a bug fix (`fix:`, `bug:`, resolved an error) move the entry to **Bug Fixes**
- If they clearly indicate a user-facing feature or improvement, move to **Delivered**
- Use judgment — do not leave value out of the report just because it lacks a ticket

**Grouping tracked work:** if Jira issue types are available, split Delivered into sub-groups (Stories/Features, Improvements). If types are not available, use a single Delivered list.

If **nothing changed**: print "No changes since `<start_date>`. Nothing to report." and still update the checkpoint.

## Step 4: Offer to share

After presenting the report, ask:

> "Want me to format this for Slack or email?"

- **Slack**: reformat as a compact message with emoji bullets, suitable for pasting into a channel update
- **Email**: wrap in a short intro line ("Here's a summary of what the team shipped this week:") and a closing line, plain text

Only reformat if the user confirms.

## Step 5: Write the checkpoint

Always update the checkpoint at the end, even on first run or when nothing changed.

```
printf 'timestamp=%s\ncommit=%s\n' "$(date -u +%Y-%m-%dT%H:%M:%SZ)" "$(git rev-parse HEAD)" > .claude/whatchanged
```

## Rules

- Write for a manager or stakeholder, not a developer — no SHAs, file names, or branch names in the report body
- Lead with shipped value; in-progress work is secondary context
- Never invent descriptions — derive them from the Jira issue summary and description only
- Always update the checkpoint, even when nothing changed
- Do not create any Jira issues or PRs — read-only only
- If the state file is corrupt or unparseable: treat it as missing (first-run path)

## Related skills

- **retro** — deeper retrospective with pattern analysis and action item creation, for the team rather than management
- **standup** — developer-focused daily summary of your own activity
- **release** — cut a GitHub release from the same change set
