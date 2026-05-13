# /deps - Audit dependencies for vulnerabilities and outdated packages, then create Jira issues

## Usage

```
/deps              # Full audit (security + outdated)
/deps --security   # Security vulnerabilities only
/deps --outdated   # Outdated packages only
```

## Flow

### 1. Setup: Read Jira state

- `jira project list --plain` → confirm the active project key

### 2. Detect package managers

Check for these manifest files in the project root and subdirectories:

| Manifest | Package manager |
|----------|----------------|
| `package.json` + `pnpm-lock.yaml` | pnpm |
| `package.json` + `yarn.lock` | yarn |
| `package.json` (neither lock file) | npm |
| `requirements.txt`, `pyproject.toml`, or `Pipfile` | pip/pip-audit |
| `Cargo.toml` | cargo |
| `go.mod` | go |
| `Gemfile` | bundler |

If no manifest files are found: "No package manifests found. Is this a new project?" and stop.

### 3. Run audits

For each detected package manager, run the appropriate commands based on the flags passed.

**Security audit commands** (skip if `--outdated` flag only):

| Package manager | Command |
|----------------|---------|
| npm | `npm audit --json` |
| yarn | `yarn audit --json` |
| pnpm | `pnpm audit --json` |
| pip | `pip-audit --format json` (fall back to `safety check --json` if pip-audit not installed) |
| cargo | `cargo audit --json` |
| go | `govulncheck ./...` |
| bundler | `bundle audit --format json` |

**Outdated check commands** (skip if `--security` flag only):

| Package manager | Command |
|----------------|---------|
| npm | `npm outdated --json` |
| yarn | `yarn outdated --json` |
| pnpm | `pnpm outdated --json` |
| pip | `pip list --outdated --format json` |
| cargo | `cargo outdated` |
| go | `go list -m -u all` |
| bundler | `bundle outdated` |

If an audit tool is not installed: note it in output, skip that check, continue with remaining checks.

If a command exits non-zero with no parseable output: show the raw error output, skip that check, continue.

### 4. Parse and prioritize findings

**Security vulnerabilities — priority by severity:**

| Severity | Priority |
|----------|----------|
| Critical | Highest |
| High | High |
| Moderate / Medium | Medium |
| Low / Info | Low |

**Outdated packages (no CVE) — priority by version delta:**

| Type | Priority |
|------|----------|
| Major version behind (e.g. 2.x → 3.x) | Medium |
| Minor version behind (e.g. 2.1 → 2.5) | Low |
| Patch only | skip — do not create a Jira issue |

Rank all findings before creating any issues.

### 5. Create Jira issues — highest priority first

Create all Highest issues, then High, then Medium, then Low.

**Summary for vulnerabilities:** `[Security] <package>@<current>: <CVE or brief description>`

**Summary for outdated packages:** `[Dependencies] Update <package> from <current> to <latest>`

**Description template:**
```
## Issue
<vulnerability description OR "Package is N major versions behind">

## Package
`<package-name>` — current: `<version>`, latest: `<latest-version>`

## CVE / Advisory
<CVE ID and link if applicable, or "None">

## Impact
<what could go wrong — use CVE description for vulnerabilities, "compatibility risk / missing features" for outdated>

## Fix
<exact command to update, e.g. `npm install package@latest`>
```

```bash
jira issue create -tTask \
  -s"[Security] package@1.0.0: CVE description" \
  -b"<description>" \
  --priority High \
  --label "dependencies,security"
```

### 6. Report

When all issues are created, print a summary:

```
Audited <N> package manager(s). Found X vulnerabilities, Y outdated packages.
Created N Jira issues.
```

List each created issue key and summary.

If all checks pass clean: "No vulnerabilities found. All packages are up to date (or within acceptable range)." — do not create any Jira issues.

## Error handling

- If `jira project list` fails: "Could not reach Jira. Is `jira` configured? Run `jira init`." and stop.
- If an audit tool is not installed: note it, skip that check, continue.
- If a command exits non-zero with no parseable output: show raw error, skip that check, continue.
- If issue creation fails: report the error, do not silently skip. Offer to retry.

## Rules

- Check every detected package manager — do not skip any
- One issue per vulnerable or outdated package, not one per audit run
- Never create an issue for a patch-only version difference
- Never create an issue for a finding that does not apply to this project
- Keep descriptions concrete: exact package name, exact version, exact fix command
