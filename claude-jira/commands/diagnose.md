---
description: Stop guessing. Think through the problem before touching anything.
argument-hint: [error message or description of what's broken]
---

You are now in diagnostic mode. Do NOT write any code, edit any files, or run any fixes yet.

The problem to diagnose: $ARGUMENTS

Follow this protocol strictly:

## Phase 1: Read Everything

Before forming any theory, gather the facts:
- Read the full error (not just the last line)
- Identify where the failure *originates* vs where it *surfaces*
- Check what changed recently (last commit, new installs, config edits)
- Note the environment: versions, env vars, file paths, permissions

State what you have found.

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

Hypotheses must be *mechanical and specific* — describe exactly what is broken in the system and why. "Maybe it's a version issue" is not a hypothesis. "Package X requires peer dependency Y@^2 but Y@3 is installed, which removed the `connect()` method" is a hypothesis.

## Phase 3: Design a Diagnostic

Pick the fastest test that distinguishes between your top two hypotheses. This could be:
- A specific log or print statement at the right location
- A direct inspection command (`which`, `env`, `ls -la`, `cat`, `curl`)
- Temporarily hardcoding a value to isolate a variable
- A minimal reproduction (strip everything until only the broken part remains)

Run the diagnostic. Update your hypothesis ranking based on the result.

## Phase 4: Fix with Confidence

Only now — with a confirmed root cause — write the fix:
- Address the root cause, not the symptom
- Make the minimum change required
- Check for side effects

Then verify it worked: run the thing, check the output, confirm the error is gone.

## Phase 5: Explain What You Found

1. What was actually wrong (root cause)
2. Why it was wrong (what condition caused it)
3. What the fix does (why it resolves the root cause)

---

If you cannot confirm a hypothesis with available information, say so clearly and ask for the specific output or file you need. It is always better to say "I need X before I can fix this" than to apply a guess.
