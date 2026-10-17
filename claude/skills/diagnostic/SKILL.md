---
name: diagnostic-thinking
description: Structured diagnosis: gather facts, form hypotheses, test, fix the confirmed root cause. Only /diagnose, or when a command names this skill. Do not trigger proactively.
---

# Diagnostic Thinking

**The problem**: Claude's default debugging mode is pattern-matching — sees an error, recalls a common fix, applies it, hopes for the best. Feels fast, often wastes more time than it saves. Engineering equivalent of shaking the TV to fix the picture.

**The goal**: Before touching anything, build a real mental model of what's wrong and *why*, then act from that understanding.

---

## The Protocol

### Phase 1: Stop. Read Everything.

Before forming any hypothesis:

- Read the **full** error message — not just the last line
- Find where in the code/config the error originates, not just where it surfaces
- Check for **multiple** errors and their order (first error is often the real one)
- Note what **changed recently** if known (recent commits, installs, config edits)
- Check environment: Node version, OS, env vars, file paths, permissions

Ask: *"What system is actually failing, and at what point does it fail?"*

---

### Phase 2: Generate Real Hypotheses

Don't go with the first plausible explanation. Generate **at least 3 candidate causes**, ranked by likelihood:

```
Hypothesis A (most likely): [specific mechanical explanation]
  Evidence for: ...
  Evidence against: ...

Hypothesis B: [alternative explanation]
  Evidence for: ...
  Evidence against: ...

Hypothesis C: [less likely but possible]
  Evidence for: ...
  Evidence against: ...
```

Good hypotheses are **mechanical and specific** — describe *what is actually broken*, not "maybe it's a version issue" or "could be a config problem".

Bad: "Maybe there's a dependency issue."
Good: "Package X expects peer dependency Y@^2.0, but Y@3.1 is installed, which introduced a breaking API change in the `connect()` function signature."

---

### Phase 3: Design a Targeted Diagnostic

Before fixing anything, **confirm which hypothesis is correct**. Pick the fastest test that distinguishes your top hypotheses:

- A log statement at the right place
- A minimal reproduction (strip everything until only the broken part remains)
- Direct inspection (`console.log`, `cat`, `env`, `which`, `ls -la`, `curl`, type-checking)
- Temporarily hardcoding a value to isolate a variable

Run the diagnostic. Let output update your hypothesis ranking.

Don't skip this even if "pretty sure" — being wrong here means applying the wrong fix, creating new problems.

---

### Phase 4: Fix with Confidence, Not Hope

Only now, with a confirmed hypothesis, write the fix. It should:

- Address the **root cause**, not the symptom
- Be the **minimum change** required
- Not break anything adjacent (check side effects)

After applying, **verify it worked** — run it, check output, confirm error gone and nothing new appeared.

---

### Phase 5: Explain What You Found

State briefly, one line each: root cause — why it happened — what the fix does and why it resolves it.

Not just for the user — writing this out confirms you actually understood the problem.

---

## Anti-Patterns to Actively Avoid

| What it looks like | Why it's a problem |
|---|---|
| Trying the most common fix without reading context | Fixes a different instance of the same-looking error |
| Making multiple changes at once | Won't know which one worked (or caused a new problem) |
| "Let me try X and see if that helps" | Debugging by lottery |
| Fixing the line that threw the error | Error is often thrown far from where it was caused |
| Assuming the same fix as last time | Context changes; same symptom can have different causes |
| Not verifying after fixing | Fix may work for the wrong reason, leaving a latent bug |

---

## When You're Stuck

If you've gone through this and still don't understand the problem:

1. **Say so clearly** — "I don't have enough information to diagnose this confidently. I need to see X."
2. **Ask a targeted question** — not "can you give me more context?" but "can you show me the output of `npm ls react` and the contents of your `package.json`?"
3. **Build a minimal reproduction** — if the real codebase is complex, strip it down until the error is isolated

Better to say "I need to understand this better before I can fix it" than run three wrong fixes in a row.

---

## `/diagnose` Command

Invoked by `/diagnose` command (`claude/commands/diagnose.md`), installed same way as rest of package — see "Testing changes" in repo's CLAUDE.md.

**Usage:**
```
/diagnose TypeError: Cannot read properties of undefined (reading 'map')
/diagnose the build fails after I added the new env variable
/diagnose   ← (no args — Claude will ask what's broken)
```

Command forces the full 5-phase protocol: gather facts → hypotheses → diagnostic → fix → explain. Cannot be shortcut.

---

## Quick Reference Checklist

Before writing any fix:

- [ ] Read the full error, not just the last line?
- [ ] Know *where in the system* the failure originates?
- [ ] Generated multiple hypotheses, not just one?
- [ ] Run a diagnostic to confirm which hypothesis is right?
- [ ] Fix addresses root cause, not the symptom?
- [ ] Will verify it worked after applying it?
