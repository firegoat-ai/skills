---
name: comparing-competitors
description: Use when you need to benchmark a provider against its competitors on the same eval prompt, or write a defensible "you vs them" narrative.
---
# Comparing competitors

A prompt's competitor benchmark comes from **sub-prompts**, not from any URL list. `get_prompt_benchmarks` assembles the whole picture — every lane and its run history; `get_report` fills in the per-run detail for specific slugs.

## Where competitor identity comes from
Competitors are sub-prompts: rows whose `parentPromptId` points at the parent prompt and whose `targetProviderId` names the competitor provider. Each sub-prompt runs the *same task* against a *different provider*. Do NOT derive competitors from a parent prompt's URL list or `additionalUrls` — sub-prompts are canonical. `get_prompt_benchmarks` already does this resolution for you.

## Tools
- `get_prompt_benchmarks({ promptId })` — the primary tool. Returns:
  - `prompt: { id, title, text, endGoal, orgDomain }`
  - `self: { provider, runCount, latest, history[] } | null` — the "you" lane (the prompt's own provider).
  - `competitors: [ { provider, runCount, latest, history[] } ]` — one lane per competitor sub-prompt.
  - Each lane's `latest`/`history[]` item is `{ slug, timestamp, score, grade, status, metrics, runType }`. `provider` is `{ id, slug, name, url }`. This mirrors the prompt-detail page the user sees. The per-lane `history[]` (ordered oldest→newest) is your trajectory source — read it for "closing the gap" vs "falling behind", not just point-in-time.
- `get_report({ slug })` — pull the full per-run detail (`metrics`, `legacy.frictionPoints`, `legacy.keyFinding`, `summary`, `recommendations`) for a specific slug once you've picked runs to contrast (e.g. each provider's `latest.slug` from `get_prompt_benchmarks`). To compare two runs, fetch each slug and diff their `metrics` yourself. Note: `score`, `grade`, and `dimensions` are no longer part of the report output.
- `list_prompts()` — to find the parent `promptId` (it lists **top-level** prompts only; competitor sub-prompts are intentionally excluded).
- `list_sub_prompts({ promptId })` — a lighter lookup when you only need the competitor lanes themselves (id, status, `targetProvider: { id, slug, name, url }`) and not their run history; prefer `get_prompt_benchmarks` once you're actually comparing runs.

## Normalize before you compare
Runs are only comparable on the **same prompt**. Even then, a prompt can run on multiple harness·model combos over time, and metrics like `time`/`cost` shift with the harness. So:
- Compare lanes within one `get_prompt_benchmarks` result (same task).
- When contrasting specific runs, prefer runs on the same harness/model; if they differ, say so rather than treating a raw time delta as a clean win.
- Lead with metric deltas (time, cost, errors, interruptions, tool calls) and the judged insights (`legacy.keyFinding`/`recommendations` via `get_report`). The report output no longer carries `score`/`grade`/`dimensions`, so build the comparison from metrics and insights, never from a grade.

## Writing the narrative
- Frame as movement, not static standings: "closed the gap from 30s to 12s", "moved from #3 to #1 on this flow" — grounded in `history[]`.
- Anchor claims to real runs: cite the competitor's and your own `latest.slug`, with the metric deltas and insights that back the claim.
- The competitor set is the org's chosen comparison, not a neutral market view; default to wins and gap-closures, and flag losses as "worth investigating" with the specific delta.
- Never invent a competitor that isn't a sub-prompt lane.
