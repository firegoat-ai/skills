---
name: suggesting-eval-prompts
description: Use when helping author a new eval prompt (its text, end goal and custom checks) or trigger an eval run, so the task actually measures the agent experience of a real end-goal.
---
# Suggesting eval prompts

An eval prompt is the task an agent is told to accomplish against a provider's product. A good prompt is the difference between meaningful AX signal and noise.

## Tools
- `list_prompts({ status?, includeArchived?, limit?, offset?, includeSubPrompts? })` — existing prompts (`{ id, title, text, endGoal, status, templateVars, createdAt }`). Check here first so you build on / avoid duplicating existing coverage. Competitor sub-prompts are excluded by default; pass `includeSubPrompts: "1"` to include them with lane identity fields `parentPromptId` / `targetProviderId`. Only `active` prompts come back by default — pass `status: "suggested"` to see prompts you or an earlier session already proposed but nobody promoted yet, or you will re-suggest them.
- `get_prompt({ promptId })` — one prompt's full configuration (`text`, `endGoal`, `frequency`, `runsPerEval`, `targetProviderId`, `additionalUrls`, `templateVars`). Read it to see exactly what an existing prompt covers.
- `get_prompt_benchmarks({ promptId })` — the prompt plus its run history and competitor lanes; use it to see how much a prompt has actually been exercised.
- `list_runs({ promptId?, reportStatus?, limit?, offset? })` — recent triggered runs and their lifecycle (`status`: `pending`/`running`/`completed`/`failed`) with the run's `report.slug` once a report lands. Use `promptId` to see a specific prompt's runs.
- `create_prompt({ text, title?, description?, endGoal?, status?, ... })` — authors a prompt and fires its first eval unless the text contains unfilled template variables or `status` is `"archived"` (the row is written without a first eval; a `parentPromptId` naming an archived prompt is refused before any write). When recommending new coverage, create at most three prompts with `status: "suggested"`; web chat renders each result as an actionable card and the dashboard keeps it under One-off prompts (db status `suggested`) until a human saves or dismisses it. Use `status: "active"` only when the user explicitly asks you to create a live prompt immediately. Put the full self-contained eval task in `text`, a short display label in `title`, and the recommendation rationale in `description`. Do not duplicate the full prompt text in prose after calling the tool.
- `request_run({ promptId, urlMap?, harnessModel?, templateArgs? })` — triggers a run of a prompt you own: creates a pending `eval_request` the queue picks up. Returns `{ runId, statusUrl, runUrl }`. It does NOT author a new prompt — use it to run one that already exists.
- `update_prompt_config({ promptId, frequency?, runsPerEval?, additionalUrls?, env?, setupCommand?, scaffold?, loginHints? })` — configure cadence, sandbox env/setup, persona instructions (`loginHints`), and `additionalUrls` (extra urls stored on the prompt — they do not create competitor lanes; lanes come from `add_competitor`). cadence (`frequency`, `runsPerEval`), `env`, `setupCommand`, `scaffold`, and `loginHints` may be set per competitor sub-prompt; `additionalUrls` stays top-level only. `env: null` clears this prompt's stored env (sub-prompts then re-inherit parent at next snapshot). Competitor lanes with null `env` / `setupCommand` inherit the parent's values at eval-request snapshot time (`resolved_env` / `resolved_setup_command`); an empty lane env map (`{}`) withholds the parent env, an empty `setupCommand` (`""`) disables it; a per-lane set overrides that field entirely (no deep merge for env). `scaffold` sets the starter codebase (react | nextjs | blank | localstack | powersync | user-docs | polyglot); it may be set per lane but does not inherit from a parent prompt. `loginHints` holds the persona instructions for the browser agent: omit=keep, `null`=clear (a scenario inherits, a top-level prompt uses its provider's), `false`=off (no persona instructions; `""` is accepted too), string=set; the response returns `loginHintsStatus` (`set | inherited | none`, same field as `list_prompts`) — the resolved status, never the text; reads (`get_prompt`, `list_prompts`, `get_trace`) return a copy with detected secrets masked, never the raw text. Read APIs return own stored env key names only. The resolved env is set as environment variables on the **coding agent's own process** (and the shells it spawns) — no `.env` file is written in the sandbox, and `setupCommand` runs earlier, as the sandbox healthcheck before the agent's environment exists, so a setup script **cannot** read the prompt's env values. Have setup write only what it can compute locally and let the agent's inherited process env supply the rest.
- `web_search` — **chat-only** (web chat and Slack; not available over MCP). The research step: search the provider's docs site for quickstarts, getting-started guides, and API reference. Use it for open questions about a provider's current docs, changelogs, or known issues.
- `web_fetch` — **chat-only** (web chat and Slack; not available over MCP). Read a promising page in full. It can ONLY fetch URLs the user pasted or that a prior `web_search` returned — never a guessed one. Always cite source URLs inline, and prefer the provider's own docs domain over third-party sources.

## From a terminal or CI
The same API ships as the `evals` CLI. When the user wants to list or trigger runs from a terminal, a script or CI, point them at the install page, https://2027.dev/evals/cli-install, then e.g. `evals runs list`, `evals runs request --prompt-id <prompt-id>` and `evals runs get --run-id <run-id>`. `evals generate-skills` writes SKILL.md files so their own coding agent can drive it.

## Research the product first
Before proposing a prompt, ground it in what the product actually offers — don't guess its capabilities. `web_search` the provider's docs for the primary onboarding flows (first API call, install the SDK, provision a resource), then `web_fetch` the quickstart to see what a realistic first-run task looks like, and derive the prompt's end-goal from a real documented flow. Cite the docs URL in `description`, not in `text`, so the user can check the flow exists. A quick search also confirms a suggested task is actually possible with the product before it becomes an eval.

## What makes a good prompt
- **State an end-goal, not steps.** "Send your first email with the API and confirm delivery" beats "call POST /emails". You're measuring whether the agent can get from zero to the outcome using only the product's docs/APIs — don't hand it the answer.
- **Make it a realistic task** a developer would actually do, so friction surfaces where real users hit it.
- **Scope it to one outcome** so the run's metrics and friction points describe one coherent task.
- **No hedges, no hints**, in `text`, `endGoal` and `checks` alike: no fallbacks, "or equivalent", "and/or" or partial credit, and no hint at the command, page or answer. Each check is one outcome that happened or didn't, e.g. `"mcp-registered": "The Chalk MCP server is registered in Claude Code."` A run that stopped at a blocker failed; no check passes for reporting it. Null a default check only when it cannot apply.
- **Use template variables** for the parts that vary per run. Declared as `templateVars` (a `string[]`) on the prompt and referenced in the text; supply values at run time via `templateArgs` (a name→value map; values are strings, max ~2000 chars, up to 50 args). Use them for things like a target region, a plan tier, or a resource name — not to leak the solution.

## Prompt status values
The prompt `status` field (plain text) is one of:
- `active` — live; eligible for scheduled runs.
- `archived` — retired; kept for history and never scheduled, but still runs when requested.
- `suggested` — AI-generated and shown in the dashboard's "Suggested prompts" section until a human saves it (→ `active`) or dismisses it.

`request_run` can trigger any prompt you own, `active`, `suggested` or `archived`. Scheduled/cron runs only fire for `active` prompts, and never for one whose `frequency` is `none` (kept for manual runs only; a sub-prompt with no cadence of its own follows its parent's, `none` included).

(Note: some UI surfaces show a *derived* status like "evaluated"/"pending" meaning "has runs yet"; that is not the stored prompt status above.)

## Triggering vs authoring
`request_run` runs an existing prompt you own. To point a run at staging/a preview, pass a `urlMap` (hostname-rewrite map, e.g. `{"docs.acme.com": "https://preview-abc123.vercel.app"}`). To pin the harness/model, pass `harnessModel: { harness, model }`. If the org isn't provider-configured, `request_run` returns an error — surface it plainly rather than retrying.

## The growth loop
Suggesting sharp prompts is how coverage grows: identify a concrete end-goal for an uncovered flow, confirm no near-duplicate exists (`list_prompts`, checking `status: "suggested"` as well as the default active list), then call `create_prompt` with `status: "suggested"` so the recommendation appears as a card and in the dashboard. If the coverage already exists as a prompt the user owns, offer to trigger it with `request_run` instead.
