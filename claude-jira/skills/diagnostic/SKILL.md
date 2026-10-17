---
name: diagnostic
description: Structured diagnosis: gather facts, form hypotheses, test, fix the confirmed root cause. Only /diagnose, or when a command names this skill. Do not trigger proactively.
---

# Diagnostic Thinking

Stop guessing. Don't touch code until root cause is confirmed.

## Phase 1: Read Everything

Before forming any theory, gather facts:
- Read full error — not just last line
- Identify where failure *originates* vs where it *surfaces*
- Check what changed recently: last commit, new installs, config edits
- Note environment: versions, env vars, file paths, permissions

State what you found before moving on.

## Phase 2: Generate Hypotheses

List **at least 3 candidate causes**, ranked by likelihood:

**Hypothesis A (most likely):** [specific mechanical explanation]
- Evidence for:
- Evidence against:

**Hypothesis B:** [alternative explanation]
- Evidence for:
- Evidence against:

**Hypothesis C:** [less likely but possible]
- Evidence for:
- Evidence against:

Hypotheses must be *mechanical and specific*. "Maybe it's a version issue" is not a hypothesis. "Package X requires peer dependency Y@^2 but Y@3 is installed, which removed the `connect()` method" is a hypothesis.

## Phase 3: Design a Diagnostic

Pick fastest test distinguishing top two hypotheses:
- A specific log or print statement at right location
- A direct inspection command (`which`, `env`, `ls -la`, `cat`, `curl`)
- Temporarily hardcoding a value to isolate a variable
- A minimal reproduction (strip everything until only broken part remains)

Run diagnostic. Update hypothesis ranking based on result.

## Phase 4: Fix with Confidence

Only now — with confirmed root cause — write the fix:
- Address root cause, not symptom
- Make minimum change required
- Check for side effects

Verify it worked: run the thing, check output, confirm error is gone.

## Phase 5: Explain What You Found

1. What was actually wrong (root cause)
2. Why it was wrong (what condition caused it)
3. What the fix does (why it resolves root cause)

## Output Format

Hypotheses as: `hypothesis — evidence for/against.` Final explanation stays prose (root cause / why / fix) — that's a narrative, not a list.

## Rules

- Don't write any code until Phase 4
- If you cannot confirm a hypothesis with available information, say so and ask for specific output or file needed
- "I need X before I can fix this" always beats applying a guess

---

## Anti-Patterns to Actively Avoid

| What it looks like | Why it's a problem |
|---|---|
| Trying the most common fix for an error message without reading context | Fixes a different instance of the same-looking error |
| Making multiple changes at once | Won't know which one worked (or caused a new problem) |
| "Let me try X and see if that helps" | Now debugging by lottery |
| Fixing the line that threw the error | Error is often thrown far from where it was caused |
| Assuming same fix as last time | Context changes; same symptom can have different causes |
| Not verifying after fixing | Fix may work for wrong reason, leaving a latent bug |

---

## When You're Stuck

If you've gone through this and still don't understand the problem:

1. **Say so clearly** — "I don't have enough information to diagnose this confidently. I need to see X."
2. **Ask a targeted question** — not "can you give me more context?" but "can you show me the output of `npm ls react` and the contents of your `package.json`?"
3. **Build a minimal reproduction** — if real codebase is complex, strip it down until error is isolated

Always better to say "I need to understand this better before I can fix it" than run three wrong fixes in a row.

---

## `/diagnose` Command

This skill is invoked by the `/diagnose` command (`claude-jira/commands/diagnose.md`), installed the same way as rest of this package — see the "Testing changes" section of the repo's CLAUDE.md.

**Usage:**
```
/diagnose TypeError: Cannot read properties of undefined (reading 'map')
/diagnose the build fails after I added the new env variable
/diagnose   ← (no args — Claude will ask what's broken)
```

Command forces full 5-phase protocol: gather facts → hypotheses → diagnostic → fix → explain. Cannot be shortcut.

---

## Quick Reference Checklist

Before writing any fix:

- [ ] Have I read the full error, not just the last line?
- [ ] Do I know *where in the system* the failure originates?
- [ ] Have I generated multiple hypotheses, not just one?
- [ ] Have I run a diagnostic to confirm which hypothesis is right?
- [ ] Is my fix addressing the root cause, not the symptom?
- [ ] Will I verify it worked after applying it?
