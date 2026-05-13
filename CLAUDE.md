# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Two parallel sets of workflow automation — both powered by Claude Code, differing only by issue tracker — that implement the same development loop: pick issue → branch → implement → PR → merge → repeat.

| Variant | Directory | Issue tracker | Commands location |
|---------|-----------|--------------|-------------------|
| Claude + Linear (primary) | `claude/` | Linear (via MCP + CLI) | `claude/commands/` |
| Claude + Jira | `claude-jira/` | Jira (via CLI) | `claude-jira/commands/` |

Linear is the daily-driver variant. The Jira variant exists so teams on Jira can adopt the same loop with the same AI tool.

## Structure

```
claude/
  commands/           # Slash command specs (.md prompt files loaded by Claude Code)
  skills/             # Reusable skill modules referenced from commands
    linear-cli/       # Linear CLI reference and best practices
    github-cli/       # GitHub CLI reference (gh)
    product-planning/ # Product thinking methodology
    diagnostic/       # Structured debugging and diagnosis protocol
    sit/              # Stop, Inspect, Think — mid-task self-audit
    bugs/             # Bug scanning methodology
    debt/             # Tech debt scanning methodology
    code-review/      # PR review methodology
    (+ more)

claude-jira/
  commands/           # Same command set, adapted to use Jira CLI instead of Linear
  skills/             # Same skill modules, adapted for Jira instead of Linear
    jira-cli/         # Jira CLI reference (instead of linear-cli)
    (rest mirrors claude/skills/)
```

Both variants install into the project's `.claude/` directory — a project uses one or the other, not both.

## How commands and skills work

**Commands** (`claude/commands/*.md`, `claude-jira/commands/*.md`) are prompt files — they are instructions written *to* Claude Code, not documentation for humans. They are prescriptive: exact CLI commands, exact MCP tool names, exact output formats. Each command is self-contained.

**Skills** (`*/skills/*/SKILL.md`) are reusable modules. A command invokes a skill by name: e.g. `Follow the diagnostic-thinking skill.` Skills have YAML frontmatter:

```yaml
---
name: skill-name
description: When to invoke this skill (used by the AI to decide relevance)
allowed-tools: Bash(linear:*), Bash(gh:*)  # optional tool restrictions
---
```

## MCP tool names (Claude + Linear only)

Commands in `claude/` reference Linear MCP tools by name. The README setup registers the server as:

```bash
claude mcp add --transport http linear-server https://mcp.linear.app/mcp
```

This produces the prefix `mcp__claude_ai_Linear__` (e.g. `mcp__claude_ai_Linear__list_issues`). If the server was registered under a different name, all references in `claude/commands/` and `claude/skills/` must be updated to match.

The Jira variant (`claude-jira/`) uses the `jira` CLI only — no MCP — because there is no Jira MCP server equivalent at the time of writing.

## Key differences between the two variants

- `claude/` commands use both MCP tools (`mcp__claude_ai_Linear__*`) and the `linear` CLI for issue management
- `claude-jira/` commands use only the `jira` CLI — no MCP
- Both reference `gh` for GitHub
- Skills are largely identical between variants; the only structural difference is `linear-cli` (in `claude/`) vs `jira-cli` (in `claude-jira/`)

## Working in this repo

- Write commands as instructions to Claude Code, not as human documentation
- Keep each command self-contained — no runtime dependencies on other commands
- When adding or changing a command in one variant, mirror the change in the other variant where the methodology applies
- Skills are shared methodology; keep them tool-agnostic where possible, or fork with a variant-specific note

## Testing changes

No test suite. Validate by installing commands in a test project and running the slash command in Claude Code:

```bash
# Linear variant
cp claude/commands/* /path/to/project/.claude/commands/
cp -r claude/skills/* /path/to/project/.claude/skills/

# Jira variant
cp claude-jira/commands/* /path/to/project/.claude/commands/
cp -r claude-jira/skills/* /path/to/project/.claude/skills/
```
