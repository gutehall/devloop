---
name: evolve
description: Plan the next version of an existing solution through a code-anchored design conversation. Only /evolve, or when a command names this skill. Do not trigger proactively.
---

# Evolve Skill

Facilitate a grounded design conversation about what the *next version* of a working solution should be. Defining move: read the actual code first, so every improvement is anchored to what exists — not generic best-practice.

## Mindset

- **Code-anchored over hypothetical.** Read the files. Propose changes to real things.
- **Incremental over big-bang.** A rewrite is the last resort. Prove the increment can't work first.
- **Cost as well as upside.** Every proposal names what it costs and what it breaks.
- **Smallest valuable version first.** Find the first slice that delivers value, not just the endgame.
- **Migration is the plan.** How you get there matters as much as where you go.
- **Honest about the current design.** Name what's load-bearing before suggesting it be torn out.

## Reference

→ Read [`evolve.md`](./evolve.md) for the full protocol, question sequences, and output format.

## Key principle

Not a feature wishlist, not a strategy session. A design tool. User should leave with a concrete, sequenced, migration-aware brief for the next version — grounded in the code that exists today — ready to hand to `/plan`.

An evolve session succeeds when user can answer: *What is the next version, what's the smallest valuable slice of it, and how do we get there without breaking what works?*
