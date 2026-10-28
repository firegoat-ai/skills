---
name: linking-to-the-dashboard
description: Use when citing a run, prompt, or trace moment in your answer — always give the user a clickable dashboard link instead of a bare slug or id.
---
# Linking to the dashboard

Every run, prompt, and trace turn has a canonical dashboard page. When you cite one in an answer, give the full clickable URL — a bare slug or uuid forces the reader to hunt. The base URL is `https://firegoat.ai/evals` (`BASE_URL` in `lib/url.ts`).

## Link formats
With `{orgDomain}` = the org's domain (e.g. `mastra.ai`):
- Prompt detail: `https://firegoat.ai/evals/{orgDomain}/prompts/{promptId}` (promptId is a uuid)
- Report overview: `https://firegoat.ai/evals/{orgDomain}/reports/{slug}`
- Full report: `https://firegoat.ai/evals/{orgDomain}/reports/{slug}/report`
- Trace with turn anchor: `https://firegoat.ai/evals/{orgDomain}/reports/{slug}/trace#turn-{N}`
- Run status (in-flight): `https://firegoat.ai/evals/{orgDomain}/runs/{runId}`
- Org home: `https://firegoat.ai/evals/{orgDomain}`

## `#turn-N` anchors
`N` is the turn's **1-based `index` field** returned by `get_trace` — NOT the 0-based `offset` parameter. Don't confuse them.

**Screenshot-merge caveat:** the dashboard trace page always interleaves browser screenshots into its turn numbering, but `get_trace` only does that with `includeBrowser: true`. So for a report that has screenshots, build `#turn-N` anchors from a trace fetched with `includeBrowser: true`, or your turn numbers will point at the wrong turns. For runs without screenshots the indices match either way.

## Where the ids come from
- `get_prompt_benchmarks` returns `prompt.{ id, orgDomain }` — prompt links are fully constructible.
- `list_prompts` / `get_prompt` return `id` but NOT `orgDomain` — use the org domain you're operating for (your conversation's org context).
- `list_runs` / `get_run` already return **absolute** `runUrl`, `statusUrl`, and (when a report exists) `report.url` (the dashboard report page) and `report.apiUrl` (the REST resource) — prefer those returned URLs over hand-constructing.
- `list_reports` / `get_report` / `get_trace` return `slug` but NOT `orgDomain` — use your conversation's org context for the link.

The org segment must be the **canonical org domain that owns the prompt/report** — the pages redirect if it's wrong, so use the orgDomain a tool returned (or the returned absolute URL), not a guess.

## Practical loop
1. Whenever you cite a run, prompt, or trace turn, construct the full URL from the formats above.
2. Get `orgDomain` from the tool response when available, otherwise from your conversation's org context.
3. Citing evidence from a specific trace turn? Link `.../trace#turn-N` (with the screenshot-merge caveat above) so the human lands on the exact moment — this is also how you let them see browser screenshots you can only read as text (see the reading-browser-screenshots skill).
