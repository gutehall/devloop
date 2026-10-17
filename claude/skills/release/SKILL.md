---
name: release
description: Build a changelog since the last tag, pick the next semver, and create a GitHub release. Only /release, or when a command names this skill. Do not trigger proactively.
---

# Release Manager

Generate a changelog, bump the version, publish a GitHub release — all from git history and merged PRs.

## Step 1: Verify environment

Run these checks before doing anything else. Stop on any failure.

**Git repo:**
```
git rev-parse --git-dir
```
Fails: "Not in a git repository." and stop.

**Current branch:**
```
git branch --show-current
```
Not `main` or `master`: warn "You're on branch `<branch>`, not main. Releases are usually cut from main. Continue? [y/n]" and wait for confirmation.

**GitHub CLI auth:**
```
gh auth status
```
Not authenticated: "GitHub CLI not authenticated. Run `gh auth login`." and stop.

## Step 2: Find baseline

```
git describe --tags --abbrev=0 2>/dev/null
```

- Tag returned: use as `<last-tag>`. Get its date with:
  ```
  git log -1 --format=%aI <last-tag>
  ```
- No tags exist: use the first commit as baseline:
  ```
  git rev-list --max-parents=0 HEAD
  ```
  Treat as a first release. Proposed version: `v0.1.0`.

## Step 3: Gather changes since baseline

**All commits since last tag:**
```
git log <last-tag>..HEAD --oneline
```
(No prior tag: use `git log --oneline`.)

**Merged PRs since last tag date:**
```
gh pr list --state merged --base main --json number,title,labels,mergedAt --limit 100
```
Filter JSON output to PRs where `mergedAt` is after the last tag date.

Both commit log and filtered PR list empty: print "No changes since `<last-tag>`. Nothing to release." and stop.

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

Anything unmatched goes under **Changes**.

For each item, prefer the PR entry (`PR #N: <title>`) over the raw commit if both refer to the same change.

## Step 5: Determine version

**Version argument passed** (`patch`, `minor`, or `major`): use it directly.

**No argument passed** — apply these rules in order:
1. Any Breaking Changes present → **major** bump
2. Any Features present → **minor** bump
3. Only fixes, chores, docs, or perf → **patch** bump

**Parse the current version** from `<last-tag>` by stripping the leading `v` (e.g., `v1.4.2` → `1.4.2`). No prior tag: start at `0.1.0`.

Apply the bump:
- **major**: increment first segment, reset minor and patch to 0
- **minor**: increment second segment, reset patch to 0
- **patch**: increment third segment only

New tag: `v<new-version>`

**Check for tag collision before proceeding:**
```
git tag -l v<new-version>
```
Tag already exists: "Tag `v<new-version>` already exists. Use `/release patch|minor|major` to force a specific bump." and stop.

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

Omit any section with no entries.

Use today's date for the release date.

Show the full draft to the user along with: "Create release `v<new-version>`? [y/n]"

Wait for confirmation. User says no or anything other than yes/y: "Release cancelled." and stop.

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
Push fails: report the error. Do not create the release. Offer to delete the local tag with `git tag -d v<new-version>`.

**Create the GitHub release:**
```
gh release create v<new-version> \
  --title "v<new-version>" \
  --notes "<changelog-content>"
```

**Print the release URL** returned by `gh release create`.

## Step 8: Update CHANGELOG.md

Check whether `CHANGELOG.md` exists in the repo root:
```
ls CHANGELOG.md 2>/dev/null
```

- **Exists:** prepend the new changelog entry (with a blank line separator) to the top of the file, below any title line if present.
- **Doesn't exist:** ask "No CHANGELOG.md found. Create one? [y/n]"
  - Yes: create the file with the new entry as its first content.
  - No: skip.

## Rules

- Never force-push tags
- Never skip the confirmation prompt in Step 6
- Always tag before creating the release — never create a release against an untagged commit
- Do not run any Linear MCP tools — this command is git and GitHub only
- Any step fails: report the exact error output and stop

## Related skills

- **linear-cli** — for managing Linear issues if release notes need to cross-reference work items

See also the `/done` command for closing the issue that tracked the release milestone.
