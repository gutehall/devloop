# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Two parallel sets of workflow automation — both powered by Claude Code, differing only by issue tracker — that implement the same development loop: pick issue → branch → implement → PR → merge → repeat.

| Variant | Directory | Issue tracker | Commands location |
|---------|-----------|--------------|-------------------|
| Claude + Linear (primary) | `claude/` | Linear (via MCP + CLI) | `claude/commands/` |
| Claude + Jira | `claude-jira/` | Jira (via Atlassian MCP + CLI) | `claude-jira/commands/` |

Linear is the daily-driver variant. The Jira variant exists so teams on Jira can adopt the same loop with the same AI tool.

## Structure

```
claude/
  commands/           # Slash command specs (.md prompt files loaded by Claude Code)
  agents/             # Sub-agents for /next initiative (context, backend, frontend, docs)
  skills/             # Reusable skill modules referenced from commands
    linear-cli/       # Linear CLI reference and best practices
    github-cli/       # GitHub CLI reference (gh)
    product-planning/ # Product thinking methodology
    diagnostic/       # Structured debugging and diagnosis protocol
    sit/              # Stop, Inspect, Think — mid-task self-audit
    bugs/             # Bug scanning methodology
    debt/             # Tech debt scanning methodology
    code-review/      # PR review methodology (loaded on L/XL diffs only)
    ship-gate/        # Shared G1–G3 quality gate for /done /pr /grind /autopilot
    (+ more)

claude-jira/
  commands/           # Same command set, adapted to use Jira CLI instead of Linear
  skills/             # Same skill modules, adapted for Jira instead of Linear
    jira-cli/         # Jira CLI reference (instead of linear-cli)
    (rest mirrors claude/skills/)
```

See [`VITALITY.md`](VITALITY.md) for core vs situational vs non-vital. Both variants install into the project's `.claude/` directory — a project uses one or the other, not both.

## How commands and skills work

**Commands** (`claude/commands/*.md`, `claude-jira/commands/*.md`) are prompt files — they are instructions written *to* Claude Code, not documentation for humans. They are prescriptive: exact CLI commands, exact MCP tool names, exact output formats.

Commands stay self-contained **relative to other commands** (a command must not say “now run /done”). If a command and a skill both exist, the command is a **thin wrapper**; the skill holds the protocol. Do not paste the same protocol into both. `/done`, `/pr`, `/grind`, and `/autopilot` invoke the **ship-gate** skill for G1–G3 rather than inlining it.

Skill `description:` frontmatter is loaded into routing context every turn. Descriptions must say when to invoke the skill and **Do not trigger proactively** unless the skill is a conventions hub owned by a consuming repo. CLI-reference skills (`linear-cli`, `jira-cli`, `github-cli`) load only when a command names them.

**Cheap subagents:** whenever a Task/Explore/subagent tool exists, locate/review greps go to the cheapest model (`cavecrew-investigator` / `cavecrew-reviewer` if those types exist, else Task/Explore on Haiku). Demand a compressed `path:line` receipt; treat it as a lead, not proof; do not re-spawn to “prove” a discarded lead. Never spawn a full-size Sonnet agent just to locate files.

This toolkit is project-agnostic. Project-specific rules live in the *consuming* repo (optional conventions skill, else `CLAUDE.md` / `AGENTS.md` / `CONTRIBUTING.md`). Never hardcode a product, verify matrix, or default base branch of `develop`.

**Skills** (`*/skills/*/SKILL.md`) are reusable modules. A command invokes a skill by name: e.g. `Follow the diagnostic-thinking skill.` Skills have YAML frontmatter:

```yaml
---
name: skill-name
description: When to invoke this skill (used by the AI to decide relevance)
allowed-tools: Bash(linear:*), Bash(gh:*)  # optional tool restrictions
---
```

## MCP tool names

Each variant registers its own MCP server:

```bash
# Linear variant
claude mcp add --transport http linear-server https://mcp.linear.app/mcp
# → tool prefix: mcp__claude_ai_Linear__

# Jira variant
claude mcp add --transport sse atlassian-server https://mcp.atlassian.com/v1/sse
# → tool prefix: mcp__claude_ai_Atlassian__
```

If a server is registered under a different name, all references in the corresponding `commands/` and `skills/` directories must be updated to match.

The Atlassian MCP gives Claude richer read access for Jira (search issues, fetch comments, follow links). The `jira` CLI is still required for writes that the MCP doesn't cover (status transitions, branch creation, sprint management).

## Key differences between the two variants

- `claude/` uses Linear MCP (`mcp__claude_ai_Linear__*`) + the `linear` CLI
- `claude-jira/` uses Atlassian MCP (`mcp__claude_ai_Atlassian__*`) + the `jira` CLI
- Both reference `gh` for GitHub
- Skills are largely identical between variants; the only structural difference is `linear-cli` (in `claude/`) vs `jira-cli` (in `claude-jira/`)

## Working in this repo

- Write commands as instructions to Claude Code, not as human documentation
- Keep each command self-contained relative to other commands — no “now run /done”
- If a command and a skill both exist, keep the command a thin wrapper and give the reusable skill a distinct name
- When adding or changing a command in one variant, mirror the change in the other variant where the methodology applies
- Skills are shared methodology; keep them tool-agnostic where possible, or fork with a variant-specific note
- Edit Linear (`claude/`) first, then mirror Jira (`claude-jira/`)

## Testing changes

No test suite. Validate by installing commands in a test project and running the slash command in Claude Code. Prefer the Core + Situational skill set from [`VITALITY.md`](VITALITY.md) — do not copy `SKILL.original.md` (gitignored).

```bash
# Linear variant (core + situational skills)
cp claude/commands/* /path/to/project/.claude/commands/
cp claude/agents/* /path/to/project/.claude/agents/   # optional: /next initiative sub-agents
for s in ship-gate linear-cli github-cli code-review prior-work product-planning diagnostic bugs debt deps release scope split sit; do
  cp -r "claude/skills/$s" /path/to/project/.claude/skills/
done

# Jira variant
cp claude-jira/commands/* /path/to/project/.claude/commands/
for s in ship-gate jira-cli github-cli code-review prior-work product-planning diagnostic bugs debt deps release scope split sit; do
  cp -r "claude-jira/skills/$s" /path/to/project/.claude/skills/
done
```
