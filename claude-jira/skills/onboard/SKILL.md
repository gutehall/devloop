---
name: onboard
description: Read a codebase and produce a structured orientation document covering architecture, entry points, patterns, and gotchas. Use this skill whenever the user runs /onboard, asks for a codebase overview, wants to understand an unfamiliar repo, or needs to produce an onboarding document.
---

# Codebase Onboarding

Read the project thoroughly and produce a structured orientation document. The goal is to give a developer everything they need to be productive on day one.

## Discovery

Start with a broad sweep before reading any source files:

1. LS the root directory — get the full picture before narrowing
2. Glob for manifest files: `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `composer.json`, `Gemfile`, `build.gradle`, `*.csproj`
3. Read `README.md` if present
4. Identify: language(s), framework(s), package manager, test framework

## Architecture mapping

Read in this order:

1. **Entry points** — `main.ts/js`, `index.ts/js`, `app.py`, `main.go`, `src/main.rs`, `cmd/*/main.go`, `server.ts/js`. Read whichever exist.
2. **Top-level directories** — list and one-sentence each
3. **Config files** — `.env.example`, `config/`, `settings.py`, `config.ts`
4. **CI/CD** — `.github/workflows/`, `Makefile`, `Dockerfile`, `docker-compose.yml`

## Pattern extraction

Read 3–5 representative source files (not just the entry point — pick files from different layers or subsystems). Extract:

| Pattern | What to look for |
|---------|-----------------|
| Imports | Module organization, barrel files, path aliases, local vs. third-party separation |
| Error handling | try/catch, Result/Either types, error middleware, panic/recover, sentinel errors |
| Config | Env vars, config objects, hardcoded values, secrets management |
| Testing | Unit vs. integration split, fixture approach, mock strategy, test helpers |
| Data layer | ORM, raw queries, HTTP clients, file I/O, caching |
| Key abstractions | Base classes, interfaces, core types, factories, DI containers |

## Gotcha hunting

This is the highest-value section. Look actively — don't skip it.

**Sources to check:**
- `.env.example` — required vars that are not obvious
- `package.json` scripts / `Makefile` targets — setup steps that aren't in README
- `CONTRIBUTING.md` — contributor-specific gotchas
- `docs/` — supplementary setup notes
- TODO / FIXME comments in source — signals of known issues or workarounds
- Unusual patterns in source that have an intent comment explaining why

**Common gotchas to surface:**
- "Run this script before starting the dev server"
- "This env var is required but not in .env.example"
- "We vendor X instead of installing it because of Y"
- "Don't use X pattern here — we use Y instead because Z"
- "This file is generated — don't edit it directly"

## Output format

```
# Codebase Orientation: <project name>

## Stack
- Language: <lang> <version if detectable>
- Framework: <framework>
- Package manager: <npm/pip/cargo/go/etc>
- Tests: <framework>

## Architecture

### Directory map
- `src/` — <purpose>
- `tests/` — <purpose>
- `config/` — <purpose>
(every top-level directory)

### Entry points
- `<file>` — <what it does>

## Patterns

### Imports
<one paragraph>

### Error handling
<one paragraph>

### Config
<one paragraph>

### Testing
<one paragraph>

## Key abstractions
- `<Class/Function/Type>` in `<file>` — <what it is>

## Gotchas
- <gotcha>
- <gotcha>

## Quick start
<minimal steps to run locally, from README + scripts>
```

Omit any section where there is genuinely nothing to report. Do not write placeholder text.

## Save mode

If `--save` was passed: after printing, write the full output verbatim to `ONBOARDING.md` at the project root. Confirm with the absolute path.

## Rules

- Read actual files. Do not infer or invent architecture.
- One sentence per directory, entry point, and abstraction.
- Gotchas section is mandatory — if you find none, look harder before writing "none found".
- Do not repeat what the README already covers clearly — reference it instead.
- Do not create ONBOARDING.md unless `--save` was explicitly passed.
