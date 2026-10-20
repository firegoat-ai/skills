---
name: spotting-friction
description: Use when you need to identify what hurt an agent's experience in an eval run and turn it into a concrete fix the dev-tool owner can ship.
---
# Spotting friction

Friction is discovered, not fetched. Scan the raw agent trace with `get_trace` — tool errors (`isError`), the agent stalling or looping (repeated `toolNames`, rising `gapAfterMs`), interruption turns — the same mechanics the reading-an-agent-trace skill uses (`toolCounts`, `totalErrors`, `totalInterruptions`, the `isError` merge-by-`tool_use_id`). A pre-graded friction list isn't evidence you've verified against this run; a trace turn is.

## Tools
- `get_trace({ slug })` — the primary source: find the turn where the agent actually hit a problem (see the reading-an-agent-trace skill for the scanning mechanics).
- `get_report({ slug })` — for `legacy.keyFinding` and `metrics` (friction rarely means anything without the metrics it explains). `get_report().legacy.frictionPoints` and `get_report().recommendations` also exist but are pre-graded, pre-written report-time fields — treat them as hints only (see below).

## The report's canned legacy.frictionPoints shape (use only as a hint)
`get_report({ slug }).legacy.frictionPoints` is a report-time snapshot (null when the report recorded none) generated once and never re-checked against what actually happened in this run. Each item: `{ id?, severity, title, desc, impact? }` — `severity` is `critical`/`major`/`minor`, `id` a shortcode like `F-01`. Treat these as a rough pointer to where problems might be, not as verified findings; your actual citation should be the trace evidence (tool name, error text, turn) you find at that location, not the `F-XX` id.

## Ignore the report's canned friction points and recommendations — derive both from the raw trace
`get_report().legacy.frictionPoints` is pre-graded and unverified against this specific run. `get_report().recommendations` (`{ id?, effort, title, desc, estimatedImpact? }`) is a legacy, pre-written field. Don't quote either as authoritative, and don't lead your narrative with `F-XX` or `R-XX` ids. Instead, derive both the friction and the fix yourself:
1. Scan `get_trace` (per the reading-an-agent-trace skill's mechanics — `toolCounts`, `totalErrors`, error-flagged `tool_use` blocks, interruption turns, stalling `gapAfterMs`) to find the actual moments where the agent had a bad experience.
2. Reason about the concrete product change from that raw evidence — what would have made *that specific tool call, error, or dead end* not happen.
3. Quote the trace turn (tool name, error text, URL, timestamp) as your evidence, not a pre-written friction point or recommendation.

You may optionally use `get_report().legacy.frictionPoints`/`.recommendations` as a rough map of where problems might be, but every claim in your final output must be grounded in something you can point to in the trace.

## Name the experience category the friction falls under
Friction usually clusters into a few recognizable buckets — setup/onboarding, speed, wasted effort, error recovery, and documentation quality. Naming the bucket a friction point falls into keeps the narrative legible — e.g. a hallucinated-URL 404 loop is an error-recovery and doc-quality problem. Ground the label in the trace evidence and the `metrics`, not in any score (the report output no longer carries `dimensions`). See the interpreting-scores skill for how to read metrics and insights as the primary signal.

## Translate friction into a shippable change
The reader is the SaaS team whose tool was evaluated. They control their **docs, APIs, SDKs, CLIs, error messages, and dashboards**. They do NOT control the agent, its prompt, or the model. So never recommend "have the agent retry" or "tell the model to X". Reframe every friction point as a product change:
- Agent installed the wrong SDK from a stale search result → fix the docs/package metadata so the canonical slug wins.
- Agent looped on a 404 for a guessed URL → return a helpful 200/redirect or a machine-readable error for agent traffic.
- Agent couldn't find the API key → add a "Copy API key" affordance / document the exact path.

## Practical loop
1. `get_trace` (with `offset: 0` for the full run, or scan `toolCounts`/`totalErrors`/`totalInterruptions` first per the reading-an-agent-trace skill) to find the turns where things went wrong — tool failures (`isError`) or the agent stalling/looping.
2. Optionally cross-check against `get_report().legacy.frictionPoints`/`.recommendations` as a hint of where the report thought problems were, but don't lead with them.
3. For each real friction moment you found in the trace, name the experience category it hurt and back it with `metrics`.
4. State the concrete doc/API/CLI/error-message change, citing the specific trace evidence (tool name, error text, turn) you found — not an `F-XX` or `R-XX` id.
