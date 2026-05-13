---
name: release
description: Generate a changelog from merged PRs and commits since the last git tag, determine the next semver version, and create a GitHub release. Use this skill whenever the user runs /release or asks to cut a release.
---

# Release Manager

Generate a changelog, bump the version, and publish a GitHub release — all from git history and merged PRs.

## Step 1: Verify environment

Run these checks before doing anything else. Stop on any failure.

**Git repo:**
```
git rev-parse --git-dir
```
If this fails: "Not in a git repository." and stop.

**Current branch:**
```
git branch --show-current
```
If not `main` or `master`: warn "You're on branch `<branch>`, not main. Releases are usually cut from main. Continue? [y/n]" and wait for confirmation.

**GitHub CLI auth:**
```
gh auth status
```
If not authenticated: "GitHub CLI not authenticated. Run `gh auth login`." and stop.

## Step 2: Find baseline

```
git describe --tags --abbrev=0 2>/dev/null
```

- If a tag is returned: use it as `<last-tag>`. Get its date with:
  ```
  git log -1 --format=%aI <last-tag>
  ```
- If no tags exist: use the first commit as baseline:
  ```
  git rev-list --max-parents=0 HEAD
  ```
  Treat this as a first release. Proposed version: `v0.1.0`.

## Step 3: Gather changes since baseline

**All commits since last tag:**
```
git log <last-tag>..HEAD --oneline
```
(If no prior tag, use `git log --oneline`.)

**Merged PRs since last tag date:**
```
gh pr list --state merged --base main --json number,title,labels,mergedAt --limit 100
```
Filter the JSON output to PRs where `mergedAt` is after the last tag date.

If both the commit log and filtered PR list are empty: print "No changes since `<last-tag>`. Nothing to release." and stop.

## Step 4: Categorize changes

Parse PR titles and commit messages against these patterns (case-insensitive):

| Category | Triggers |
|----------|----------|
| Breaking Changes | `BREAKING`, `breaking change`, `!:` anywhere in the title |
| Features | title starts with or contains `feat:`, `feature:`, `add `, `new ` |
| Bug Fixes | title starts with or contains `fix:`, `bug:`, `patch:` |
| Performance | title starts with or contains `perf:`, `performance:` |
| Documentation | title starts with or contains `docs:`, `doc:` |
| Chores | title starts with or contains `chore:`, `refactor:`, `test:`, `ci:`, `build:`, `deps:` |

Anything not matched goes under **Changes**.

For each item, prefer using the PR entry (`PR #N: <title>`) over the raw commit if both refer to the same change.

## Step 5: Determine version

**If a version argument was passed** (`patch`, `minor`, or `major`): use it directly.

**If no argument was passed**, apply these rules in order:
1. Any Breaking Changes present → **major** bump
2. Any Features present → **minor** bump
3. Only fixes, chores, docs, or perf → **patch** bump

**Parse the current version** from `<last-tag>` by stripping the leading `v` (e.g., `v1.4.2` → `1.4.2`). If no prior tag, start at `0.1.0`.

Apply the bump:
- **major**: increment first segment, reset minor and patch to 0
- **minor**: increment second segment, reset patch to 0
- **patch**: increment third segment only

New tag: `v<new-version>`

**Check for tag collision before proceeding:**
```
git tag -l v<new-version>
```
If the tag already exists: "Tag `v<new-version>` already exists. Use `/release patch|minor|major` to force a specific bump." and stop.

## Step 6: Draft changelog

Format the changelog as:

```markdown
## v<new-version> — YYYY-MM-DD

### Breaking Changes
- PR #N: <title>

### Features
- PR #N: <title>
- commit: <short-sha> <message>

### Bug Fixes
- PR #N: <title>

### Performance
- PR #N: <title>

### Documentation
- PR #N: <title>

### Chores
- PR #N: <title>

### Changes
- PR #N: <title>
```

Omit any section that has no entries.

Use today's date for the release date.

Show the full draft to the user along with: "Create release `v<new-version>`? [y/n]"

Wait for confirmation. If the user says no or anything other than yes/y: "Release cancelled." and stop.

## Step 7: Create the release

Execute in order. Stop and report on any failure — do not proceed to the next step.

**Tag the commit:**
```
git tag v<new-version>
```

**Push the tag:**
```
git push origin v<new-version>
```
If push fails: report the error. Do not create the release. Offer to delete the local tag with `git tag -d v<new-version>`.

**Create the GitHub release:**
```
gh release create v<new-version> \
  --title "v<new-version>" \
  --notes "<changelog-content>"
```

**Print the release URL** returned by `gh release create`.

## Step 8: Update CHANGELOG.md

Check whether a `CHANGELOG.md` exists in the repo root:
```
ls CHANGELOG.md 2>/dev/null
```

- **If it exists:** prepend the new changelog entry (with a blank line separator) to the top of the file, below any title line if one is present.
- **If it does not exist:** ask "No CHANGELOG.md found. Create one? [y/n]"
  - If yes: create the file with the new entry as its first content.
  - If no: skip.

## Rules

- Never force-push tags
- Never skip the confirmation prompt in Step 6
- Always tag before creating the release — do not create a release against an untagged commit
- If any step fails, report the exact error output and stop

## Related skills

- **jira-cli** — for managing Jira issues if release notes need to cross-reference work items
- **done** — for closing the issue that tracked the release milestone
