# /autopilot - Allowlisted autonomous one-cycle: pick → implement → ship

`/grind`, but **hard-gated to issues labelled `auto-claude`**. Built for an unattended Claude Code instance: it may only ever touch work a human has explicitly opted in by adding the `auto-claude` label. Everything else is invisible to it.

```
/loop /autopilot project       # drain the allowlisted queue in a project, one issue per cycle
/loop /autopilot issue          # one allowlisted issue per branch per cycle
```

Run bare (`/autopilot project`) to do exactly one cycle and stop.

> **Safety contract:** the automated instance is configured to run **only** `/autopilot` — never `/grind` or `/next`. `/autopilot` never reads, branches from, comments on, transitions, or implements any issue that does not carry the `auto-claude` label. If it ever picks up an issue without the label, it **stops the loop** rather than proceeding.

## How it runs

**Follow `/grind` exactly** — same scope resolution, same cycle (pick → read → prior-work → branch → implement → ship-gate → ship → CI → return to base), same loop control, same self-paced timing. Apply these deltas:

### 1. Queue filter (every issue query)

Add `--label "auto-claude"` to every `linear issues` query in the pick step, in both scopes:

```bash
linear issues --project "<project>" --status "Ready for build" --label "auto-claude"   # project mode
linear issues --status "Ready for build" --label "auto-claude"                          # issue mode
```

Never fall back to unlabelled issues, blocked issues, or other statuses. Empty queue → **STOP LOOP** (clean finish):
`autopilot: no issues in "Ready for build" labelled auto-claude. Stopping loop.`

### 2. Hard gate — verify the label after `linear issue show <id>`

Before doing anything else with the picked issue, confirm it actually carries the `auto-claude` label. If it does **not** (stale/unexpected filter result) → **STOP LOOP**, touch nothing:
`autopilot: <id> is not labelled auto-claude — refusing to work it. Stopping loop.`

### 3. Skip actions keep the label

When `/grind` would move a skipped issue to Backlog (non-code, ambiguous, already-solved, in-flight), keep `auto-claude` in place so a human can re-queue it:

```bash
linear issue update <id> --status "Backlog" --label "auto-claude" --label "needs-human"
```

### 4. Merge — never direct-merge

An unattended instance never runs a direct `gh pr merge`. In `/grind` step 7, the only change:

- **Repo does not allow auto-merge (`--auto` rejected)** → **STOP LOOP**, leave the PR open, print `autopilot: cannot arm auto-merge on <PR> — review and merge manually. Stopping loop.` Never fall back to a direct merge.

Every other CI/merge branch (fail, no checks, armed-awaiting-approval, merged-immediately) is identical to `/grind`.

### 5. Receipts

All progress/stop lines are prefixed `autopilot:` instead of `grind:`. Success receipt:
```
autopilot ✓ ISSUE-12 merged (PR #214). Next cycle.
```

## Stop conditions

Every `/grind` stop condition applies, plus:

| Condition | Action |
|-----------|--------|
| No allowlisted (`auto-claude`) issues in Ready for build | STOP LOOP (clean finish) |
| Picked issue is missing the `auto-claude` label | STOP LOOP, touch nothing |
| Auto-merge not allowed by repo settings | STOP LOOP, PR left open — never direct-merge |

Never: prompt the user, work an issue without the `auto-claude` label, pick blocked work, guess work type, force-push, run a direct `gh pr merge`, or `linear issue close` before merge.
