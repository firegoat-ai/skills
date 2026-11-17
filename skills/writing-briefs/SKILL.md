---
name: writing-briefs
description: >-
  How to write a Firegoat brief, the short published write-up over a set of eval runs: bottom line, grade table, evidence linked to trace turns, fixes, confidence. Use when asked to write, draft, update or publish a brief, a period report, or a summary of a tier eval or a batch of runs for the org to read.
---

# Writing briefs

A brief is a short page the whole org reads at `/{org}/briefs/{slug}` once published. Lead with the answer. A reader who stops after the first paragraph should still know the result.

## Tools
- `create_brief({ periodKind, anchorDate, endDate?, title?, promptId?, runIds? })` makes a draft over a period (`day`, `week`, `month`, `custom`) and freezes runs into the brief's scope, averaging their metrics. Scope it to the runs the brief is about:
  - `promptId`: only that prompt's runs, including its scenario and competitor sub-prompts. A sub-prompt id is refused; to cover one scenario, use `runIds`.
  - `runIds`: exactly those runs (the ids `get_run` / `list_runs` return). A run outside the period or the org fails, and the error names it.
  - Both: every run in `runIds` must belong to the prompt.
  - Neither: every published run in the org in that period, related or not. Pass one of the two for a tier eval or a single-prompt brief, and check the frozen run count in `list_briefs`.
- `update_brief({ slug, body?, title?, status? })` replaces the whole markdown body. Set the title with `title`, not with a `#` heading in the body.
- `publish_brief({ slug })` publishes the current body as the next version. It refuses while a run citation doesn't resolve.
- `get_brief({ slug, version? })`, `list_briefs({ status? })`, `list_brief_versions({ slug })` read what exists.

## Shape
Keep it to one screen. Use these sections, in this order:

1. **Bottom line.** Two or three sentences: what was measured, the result, and the one thing to do about it.
2. **Grade table.** One row per product, lane or prompt: the grade (score, checks passed, or tier), runs counted, and a few words on why. For an AX tier list placement, use the four criteria and the discovery column from placing-on-the-ax-tier-list.
3. **Evidence.** For every claim in the table, cite the run inline with `[[mention:<report-slug>]]`, and link the exact moment with the trace URL `.../reports/{slug}/trace#turn-N` (see linking-to-the-dashboard for the URL and the turn numbering). Quote the agent's own words where they show the problem.
4. **Fixes.** What the product owner can change, most impact first, each tied to the evidence above.
5. **Confidence.** How many runs, which harness and model, and what the result rests on: traces, docs, or a check you ran. Say what would change the conclusion.

## Rules
- Only claim what a cited run shows. If the brief's scope holds runs you didn't read, leave them out of the grade table rather than averaging them in silently.
- `[[mention:<report-slug>]]` is the only run citation `publish_brief` checks. Use it, then add the trace link for the specific turn.
- Draft, read it back with `get_brief`, then publish. Publishing again creates a new version; earlier versions stay readable.
