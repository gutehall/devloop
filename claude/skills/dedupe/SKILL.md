---
name: dedupe
description: Find duplicate Linear issues, cluster them, and walk the user through resolving them. Only /dedupe, or when a command names this skill. Do not trigger proactively.
---

# Duplicate Detection

Find issues describing the same work, group into clusters, pick a canonical issue per cluster, let user decide how to collapse each group. Never merge or cancel without confirmation.

## Mindset

Two issues are duplicates when acting on one makes the other redundant — same bug, same feature, same fix. Don't flag issues that merely share a topic. Test: if you closed one, would the other still represent real outstanding work? If yes, they're related, not duplicate.

False positives waste time and risk losing tracked work. Bias toward precision — when unsure, label a pair "possible" not "duplicate", let user judge.

## Loading the workspace

Fetch in parallel:

1. `mcp__claude_ai_Linear__list_teams` — active team
2. `mcp__claude_ai_Linear__list_issues` — all issues to compare

By default compare **open work**: Backlog, Todo, In Progress. Exclude Done and Canceled — already resolved, re-opening that history rarely wanted. Honor flags that widen the set:

| Flag | Set compared |
|------|--------------|
| (none) | Backlog, Todo, In Progress |
| `--all` | every status including Done/Canceled |
| `--project "<name>"` | only issues in that project |
| `--label <name>` | only issues carrying that label |

If issue list is large (>200), tell user the count and that comparison is title/description-based, then proceed.

## Cheap pre-filter before full comparison

Full pairwise semantic comparison is O(n²) — on 250 issues that's ~31k pairs, most of them obviously unrelated. Before running semantic judgment, bucket issues with cheap heuristics and only compare within a bucket:

- Same label(s), or
- Same project/component, or
- Title shares at least one significant word (ignore stopwords like "fix", "add", "the")

Issues that share no bucket with anything are almost never duplicates — skip full comparison for them (they surface, if at all, as unclustered singletons; do not force a pairing). Within a bucket, apply the full semantic signal table below. This cuts the comparison set from all-pairs to same-bucket pairs, which is normally a small fraction on a real backlog.

If the workspace is small (<40 open issues), skip the pre-filter — the full O(n²) pass is cheap enough there and bucketing adds overhead for no savings.

## Detecting duplicates

Compare issues pairwise (within a bucket from the pre-filter above). Signals, strongest first:

| Signal | Weight |
|--------|--------|
| Near-identical title (same words, reordered or reworded) | High |
| Description describes the same symptom, file, or feature | High |
| Same error message, stack trace, or reproduction steps | High |
| Same labels + same area/component + overlapping title terms | Medium |
| One issue's title is a subset of another ("Fix login" vs "Fix login redirect bug") | Medium — may be parent/child, not duplicate |
| Shared topic only, different specifics | Low — usually NOT a duplicate |

Read titles/descriptions semantically — "App crashes on logout" and "Logout throws null pointer" are the same bug despite no shared keywords. "Add dark mode" and "Fix dark mode toggle" are different work.

Classify each pair/cluster by confidence:

- **Duplicate** — clearly same work. Safe to recommend collapsing.
- **Possible** — overlapping, needs a human call. Present, don't recommend.
- **Related** — same area but distinct work. Suggest linking, not merging.

## Picking the canonical issue

Within a duplicate cluster, canonical is the one to keep. Prefer, in order:

1. Most complete — real description, acceptance criteria, estimate
2. Most activity — comments, assignee, in-progress status, linked PR/branch
3. Oldest — the original, if above tie

Everything else in cluster collapses into canonical.

## Presenting clusters

Group findings by confidence, most actionable first. Specific — IDs and titles, not counts alone:

```
Duplicates (2 clusters)

  Cluster A — keep FIN-12
    FIN-12  "Login redirect loops on expired session"   (has description, assignee, in progress)
    FIN-47  "Infinite redirect when token expired"       (no description, backlog)
    → same bug. Recommend: cancel FIN-47, link to FIN-12.

  Cluster B — keep FIN-3
    FIN-3   "Add CSV export to billing"
    FIN-29  "Export invoices as CSV"
    → same feature. Recommend: merge FIN-29 into FIN-3.

Possible (1 cluster) — needs your call
    FIN-8   "Dashboard slow to load"
    FIN-31  "Optimize dashboard queries"
    → FIN-31 may be the fix for FIN-8, or a separate task.

Related (linking only)
    FIN-5 / FIN-19 — both touch auth, different work. Offer to link as related.
```

## Acting on a cluster

Linear has no merge or delete via API or CLI — true merge (moving comments, sub-issues, relations) only happens in the Linear UI. Offer these concrete actions per cluster, apply only the confirmed one:

**1. Collapse (cancel duplicate, keep canonical)** — default for clear duplicates:
- Copy any context the duplicate has that canonical lacks into canonical:
  - `mcp__claude_ai_Linear__save_issue` to append to canonical's description, or
  - `mcp__claude_ai_Linear__save_comment` to add duplicate's useful notes
- Comment on duplicate linking canonical: `mcp__claude_ai_Linear__save_comment` → "Duplicate of FIN-12. Collapsed via /dedupe."
- Cancel duplicate: `linear issue update FIN-47 --status "Canceled"`

**2. True merge in Linear UI** — when duplicate has comments, sub-issues, or relations worth preserving:
- Don't cancel. Tell user to merge in Linear: open duplicate, "⋯ → Merge into issue", target canonical.
- Provide both issue identifiers for speed.

**3. Link as related** — for "related", not duplicate:
- Add cross-referencing comment on each via `mcp__claude_ai_Linear__save_comment`.

**4. Skip** — leave cluster untouched.

Present per cluster:

```
Cluster A — keep FIN-12, collapse FIN-47
[Enter] Collapse (cancel FIN-47, link to FIN-12)
[m] Merge in UI instead   [l] Link as related   [s] Skip   [q] Quit
```

## Output

After all clusters:

```
Reviewed N clusters. Collapsed M, merged-in-UI K, linked L, skipped S.
```

List each collapsed duplicate (`FIN-47 → FIN-12`) and any cluster left for manual UI merge.

## Error handling

- `save_issue` / `save_comment` / status update fails: print error, offer retry or skip — never silently continue, never leave a duplicate canceled without its cross-link comment.
- MCP unreachable: "Could not reach Linear. Check your MCP connection with `/mcp`." and stop.
- Canonical pick ambiguous (true tie): present cluster, ask user which to keep before acting.

## Rules

- Never cancel or modify an issue without explicit confirmation for that cluster.
- Always preserve context — copy useful description/comments into canonical before canceling a duplicate.
- Never use delete; Linear has no API delete and cancellation is reversible. Cancellation is the strongest action this skill takes.
- Duplicate with comments/sub-issues/relations: prefer UI merge over collapse so nothing is orphaned.
- Bias toward "possible" over "duplicate" when uncertain — let human decide.

## Related skills

- **linear-cli** — issue status updates and CLI reference
- **triage** — clean up metadata on surviving issues after deduping
- **scope** — broader project audit for gaps and quality
