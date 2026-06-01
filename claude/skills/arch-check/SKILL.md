---
name: arch-check
description: >
  Run the finctl-arch-agent fast static CDK architecture scan on demand and
  present findings grouped by pillar and severity, with clickable file:line
  references. Local-first — no cloud calls, no AWS credentials, no model. Honors
  the committed baseline and reports only NEW findings by default. Use when the
  user types /arch-check, or asks to "scan the CDK", "check architecture",
  "run arch-agent", "review my stacks for security/cost/reliability issues", or
  wants the on-demand alternative to the git pre-push hook. Only applies in a
  repo that vendors the finctl-arch-agent wrapper (scripts/arch-agent.sh); in any
  other repo this skill should stop and say so.
---

# /arch-check — on-demand CDK architecture scan

Interactive, local-first wrapper around `finctl-arch-agent` (FIN-2614). It is the
manual counterpart to the git pre-push hook: same scan, same baseline, run when
the developer asks.

## 0. Guard — is this repo wired for arch-agent?

This skill is generic devloop tooling but only works where the finctl-arch-agent
wrapper is vendored. **Before doing anything else**, check for the wrapper:

```bash
test -x scripts/arch-agent.sh && echo present || echo absent
```

If it prints `absent` (or the file is missing): stop. Tell the user this repo
doesn't have the finctl-arch-agent wrapper (`scripts/arch-agent.sh`) and that
`/arch-check` only applies in repos that vendor it (e.g. finctl-core). Do not try
to install anything.

## 1. Decide the scan target

- If the user passed a path argument, scan that path.
- Otherwise, default to the **changed CDK files** vs the working tree:
  ```bash
  git diff --name-only HEAD -- 'src/stacks/**' 'src/app.ts'
  git diff --name-only --cached -- 'src/stacks/**' 'src/app.ts'
  ```
  If nothing changed, fall back to scanning `src` (the wrapper's default).

The wrapper scans a directory (`src` by default), not a file list, so the
changed-file detection is used to decide *whether* to scan and to tell the user
what prompted it — not to narrow the scan itself. Keep the scan target `src` so
finding paths match the baseline.

## 2. Run the scan (baseline-aware, no cloud)

```bash
ARCH_AGENT_ENFORCE_BASELINE=1 ARCH_AGENT_FORMAT=json scripts/arch-agent.sh
```

- `ARCH_AGENT_ENFORCE_BASELINE=1` → only findings absent from
  `.finctl-arch-agent-baseline.json` are treated as new.
- `--no-llm` is forced by the wrapper; **never** set `ARCH_AGENT_*` escalation /
  classify flags here — this skill is the fast, model-free path. (Tiered local /
  cloud classification is a separate, opt-in flow.)
- Reports land in `tools/arch-agent/out/findings.json` and `findings.md`.

To show *only new* findings (default), diff the JSON findings against the
baseline fingerprints `(id, file, line)` in `.finctl-arch-agent-baseline.json`.
If the user asks for "all findings", read the full `findings.json` instead.

## 3. Present the results

Read `tools/arch-agent/out/findings.json` and summarize:

- Group by **pillar** (the `category` field: Security, Cost, Performance
  Efficiency, Reliability, Operational Excellence, Sustainability, Best
  Practices), then by **severity** (Critical > High > Medium > Low).
- For each finding print one line: `severity · [RULE-ID] title — file:line`.
  Render `file:line` so it is clickable in the terminal (relative to the scanned
  dir, e.g. `src/stacks/foundation-stack.ts:135`).
- Lead with a one-line count: total new findings and the worst severity.
- If there are zero new findings, say so plainly.

Keep it terminal-friendly and scannable. Do not invent fixes unless asked — this
is a review, not a refactor.
