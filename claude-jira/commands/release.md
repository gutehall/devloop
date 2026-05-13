# /release - Create a GitHub release with generated changelog

Generates a changelog from merged PRs and commits since the last git tag, determines the next semver version, and creates a GitHub release.

## Usage

```
/release            # Auto-detect version bump from changes
/release patch      # Force patch bump
/release minor      # Force minor bump
/release major      # Force major bump
```

## Flow

### 1. Find the last release

```bash
git tag --sort=-v:refname | head -1   # Latest semver tag
git log <last-tag>..HEAD --oneline --no-merges
```

If no tags exist: treat the entire history as the changelog.

### 2. Fetch merged PRs since last tag

```bash
gh pr list --state merged --json number,title,author,mergedAt,labels --limit 100
```

Filter to PRs merged after the last tag's commit date.

### 3. Categorize changes

Group PRs and commits into changelog sections based on title prefixes and labels:

| Category | Signals |
|----------|---------|
| Breaking Changes | `breaking`, `major` label, `!` in conventional commit |
| Features | `feat:`, `feature` label, Story issue type |
| Bug Fixes | `fix:`, `bug` label, Bug issue type |
| Performance | `perf:` |
| Dependencies | `deps:`, `chore(deps):`, `dependencies` label |
| Internal | `chore:`, `refactor:`, `ci:`, `test:` |

### 4. Determine next version

Parse the last tag as semver (`vMAJOR.MINOR.PATCH`):
- If `major` argument passed or Breaking Changes exist → bump MAJOR, reset MINOR and PATCH to 0
- If `minor` argument passed or Features exist → bump MINOR, reset PATCH to 0
- If `patch` argument passed or only Bug Fixes / Dependencies / Internal → bump PATCH
- If no changes detected → bump PATCH

Show the proposed version and ask: "Release as vX.Y.Z? [Enter to confirm / type different version]"

### 5. Generate changelog

```markdown
## vX.Y.Z — YYYY-MM-DD

### Breaking Changes
- PR #N: description

### Features
- PR #N: description

### Bug Fixes
- PR #N: description

### Dependencies
- PR #N: description
```

Omit empty sections.

### 6. Create the GitHub release

```bash
gh release create vX.Y.Z \
  --title "vX.Y.Z" \
  --notes "<changelog content>"
```

Print the release URL.

## Notes

- If no PRs are found (commits only): generate changelog from commit messages
- Jira issue keys in PR titles (e.g. `PROJ-42`) are preserved in the changelog for traceability
- Do not create the release without user confirmation of the version number
