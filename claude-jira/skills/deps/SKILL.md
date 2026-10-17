---
name: deps
description: Audit dependencies for vulnerabilities and outdated packages, then create Jira issues. Only /deps, or when a command names this skill. Do not trigger proactively.
---

# Dependency Auditor

Detect all package managers in project, run security and outdated checks, create one Jira issue per finding, ordered by severity.

## Setup

Before auditing:

1. `jira project list --plain` — confirm active project key
2. Note project key (e.g. `PROJ`) for issue creation

Skip this if project key already known from earlier in the session.

## Package manager detection

Check for manifest files in project root and subdirectories:

| Manifest | Package manager |
|----------|----------------|
| `package.json` + `pnpm-lock.yaml` | pnpm |
| `package.json` + `yarn.lock` | yarn |
| `package.json` (no lock file) | npm |
| `requirements.txt`, `pyproject.toml`, or `Pipfile` | pip/pip-audit |
| `Cargo.toml` | cargo |
| `go.mod` | go |
| `Gemfile` | bundler |

If no manifests found: "No package manifests found. Is this a new project?" and stop.

## Audit commands

### Security

| Package manager | Command |
|----------------|---------|
| npm | `npm audit --json` |
| yarn | `yarn audit --json` |
| pnpm | `pnpm audit --json` |
| pip | `pip-audit --format json` (fall back to `safety check --json`) |
| cargo | `cargo audit --json` |
| go | `govulncheck ./...` |
| bundler | `bundle audit --format json` |

### Outdated

| Package manager | Command |
|----------------|---------|
| npm | `npm outdated --json` |
| yarn | `yarn outdated --json` |
| pnpm | `pnpm outdated --json` |
| pip | `pip list --outdated --format json` |
| cargo | `cargo outdated` |
| go | `go list -m -u all` |
| bundler | `bundle outdated` |

If a tool is not installed: note it, skip check, continue.
If a command exits non-zero with no parseable output: show raw error, skip, continue.

## Priority mapping

### Security vulnerabilities

| Severity | Priority |
|----------|----------|
| Critical | Highest |
| High | High |
| Moderate / Medium | Medium |
| Low / Info | Low |

### Outdated packages (no CVE)

| Type | Priority |
|------|----------|
| Major version behind (e.g. 2.x → 3.x) | Medium |
| Minor version behind (e.g. 2.1 → 2.5) | Low |
| Patch only | skip — do not create a Jira issue |

## Creating Jira issues

Rank findings first, create in order: Highest → High → Medium → Low.
Priority order keeps backlog correctly sorted. Ask if issues go under existing epic or unassigned.

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

## Output Format

One line per finding when listing to the user:

`package@version: [severity] problem — fix.`

Then the summary block once all issues are created:

```
Audited <N> package manager(s). Found X vulnerabilities, Y outdated packages.
Created N Jira issues.
```

List each created issue key and summary.

If clean: "No vulnerabilities found. All packages are up to date (or within acceptable range)." — do not create any Jira issues.

## Rules

- Run every detected package manager — skip none
- One issue per vulnerable or outdated package
- Never create issue for patch-only version difference
- Keep descriptions concrete: exact package name, exact version, exact fix command

## Related skills

- **jira-cli** — full CLI reference for issue management
- **bugs** — for scanning codebase itself for code-level bugs
