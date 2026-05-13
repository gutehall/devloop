# /onboard - Codebase orientation

Read the current project and produce a structured orientation document covering architecture, entry points, patterns, and gotchas.

## Usage

```
/onboard            # Print orientation to the conversation
/onboard --save     # Same, and write ONBOARDING.md to project root
```

## Step 1: Discover project structure

1. List the root directory to get the full file and directory list
2. Find manifest files: `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `composer.json`, `Gemfile`, `build.gradle`, `*.csproj`
3. Read `README.md` if it exists
4. Detect: language(s), framework(s), package manager, test framework

## Step 2: Map the architecture

Read key files to understand structure:

- Main entry points: `main.ts`, `main.js`, `index.ts`, `index.js`, `app.py`, `main.go`, `src/main.rs`, `cmd/*/main.go`, `server.ts`, `server.js`
- Top-level directories — infer purpose from names and content
- Config files: `.env.example`, `config/`, `settings.py`, `config.ts`, `config.js`
- CI/CD: `.github/workflows/`, `Makefile`, `Dockerfile`, `docker-compose.yml`

For each top-level directory: one-sentence description of what it contains.

## Step 3: Identify patterns

Read 3–5 representative source files and extract:

- **Imports** — how modules/packages are imported and organized
- **Error handling** — exceptions, Result types, error callbacks, panic/recover
- **Config management** — env vars, config files, hardcoded values
- **Testing** — unit vs. integration, fixture style, mock approach
- **Key abstractions** — base classes, interfaces, or core types everything else builds on
- **Data layer** — ORM, raw SQL, HTTP API calls, file I/O

## Step 4: Find gotchas

Look for things that would surprise a developer on day one:

- Non-obvious setup steps — scripts in `package.json`, `Makefile` targets, required env vars missing from README
- Known quirks — unusual patterns, workarounds, non-idiomatic code with intent comments
- Important conventions not captured in a README
- Check `.env.example` for required variables
- Check `CONTRIBUTING.md` and `docs/` for setup notes
- Scan for `TODO` and `FIXME` comments that signal known issues

## Step 5: Output

Print the orientation in this exact format:

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
(list every top-level directory)

### Entry points
- `<file>` — <what it does>

## Patterns

### Imports
<how imports are organized — local vs. third-party, barrel files, path aliases, etc.>

### Error handling
<how errors are handled — try/catch, Result types, error middleware, etc.>

### Config
<how config and env vars are managed>

### Testing
<testing approach, frameworks, and conventions>

## Key abstractions
- `<Class/Function/Type>` in `<file>` — <what it is and why it matters>

## Gotchas
- <thing that would surprise a developer>
- <required setup step not covered in README>
- <known quirk or workaround>

## Quick start
<minimal steps to run the project locally, derived from README + scripts>
```

If `--save` was passed: write this output verbatim to `ONBOARDING.md` at the project root. Confirm with: `Saved to ONBOARDING.md`.

## Rules

- Read actual source files — do not invent architecture
- Keep descriptions short: one sentence per directory, entry point, and abstraction
- The Gotchas section is the most valuable — invest time here
- If README already covers something well, note "see README" rather than repeating it
- If a section has nothing to report (e.g. no CI/CD found), omit it rather than writing "N/A"
