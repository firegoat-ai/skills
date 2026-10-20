---
name: harnesses-and-models
description: Use when a question involves which coding agent (harness) or model ran an eval, comparing runs across harnesses/models, or pinning a harness+model when triggering a run.
---
# Harnesses and models

Every run executes on a **harness · model** combo: the harness is the coding agent (the product being measured is used *through* it), the model is the LLM inside it. The same prompt can behave very differently across combos, so knowing which combo produced a run is load-bearing for any comparison.

## The registry
The only combos the queue accepts:
| Harness | Models |
|---|---|
| `claude-code` | claude-opus-5-5 (Opus 5.5), claude-opus-5 (Opus 5), claude-opus-4-8 (Opus 4.8), claude-opus-4-6 (Opus 4.6), claude-sonnet-4-6 (Sonnet 4.6), claude-sonnet-5-5 (Sonnet 5.5), claude-sonnet-5 (Sonnet 5), claude-haiku-5-5 (Haiku 5.5), claude-haiku-4-5-20251001 (Haiku 4.5), claude-fable-5-1 (Fable 5.1), claude-fable-5 (Fable 5) |
| `codex` | openai/gpt-5.5 (GPT-5.5), openai/gpt-5.6 (GPT-5.6), openai/gpt-5.6-sol (GPT-5.6 Sol), openai/gpt-5.6-terra (GPT-5.6 Terra), openai/gpt-5.6-luna (GPT-5.6 Luna) |
| `opencode` | openai/gpt-5.5 (GPT-5.5), openai/gpt-5.6 (GPT-5.6), anthropic/claude-opus-4-8 (Opus 4.8), anthropic/claude-opus-4-6 (Opus 4.6), xai/grok-4.3 (Grok 4.3), xai/grok-4.5 (Grok 4.5), xai/grok-4.6 (Grok 4.6), xai/grok-4.7 (Grok 4.7) |
| `openclaw` | openai/gpt-5.5 (GPT-5.5), openai/gpt-5.6 (GPT-5.6) |
| `cursor-cli` | cursor/auto (Auto) |
| `fx` | vercel_ai_gateway/anthropic/claude-sonnet-4.5 (Sonnet 4.5), vercel_ai_gateway/anthropic/claude-opus-4.6 (Opus 4.6) |
| `v0` | v0/v0-mini (v0 Mini), v0/v0-pro (v0 Pro), v0/v0-max (v0 Max) |
| `grok-build` | grok-build-0.1 (Grok Build 0.1) |

Default when nothing is pinned: `claude-code` + `claude-opus-4-8`.

## Historical quirk: null harness
Older runs recorded a model but no harness. Resolution rule: a null harness on a model that claude-code can run means it WAS the claude-code default; only models claude-code can't run (gpt-5.5, gpt-5.6, cursor/auto, Kimi, Grok) stay genuinely unknown.

## Where harness · model is exposed
The run-centric tools carry the submitted combo directly: `list_runs` and `get_run` return top-level `harness` and `model` fields (the config the run was triggered with) — that's the reliable source. The report-centric tools do NOT: `get_report` may carry a model inside `providerMeta.model` but no harness, and `list_reports` / `get_prompt_benchmarks` history items return neither. So if a user asks "which harness/model ran this?": for a run, read `list_runs`/`get_run` `harness`/`model`; for a report slug only, try `get_report().providerMeta.model`, else say the report tools don't expose the harness and point them at the run's dashboard report page (see the linking-to-the-dashboard skill) rather than guessing.

## Pinning at trigger time
`request_run({ promptId, harnessModel: { harness, model } })` pins the combo — it's validated against the registry above and unknown combos are rejected. Omit `harnessModel` to let the queue use the default.

## Comparability
A prompt accumulates runs across multiple harness·model combos over time. Metrics like `time` and `cost` shift with the harness — there is no normalization. Compare metrics within the same combo when you can; when combos differ, say so explicitly instead of presenting a raw delta as a clean win. A lane's `history[]` from `get_prompt_benchmarks` can silently mix combos, so cross-check the combo (via `list_runs`/`get_run`) before reading a trajectory as a clean trend.

## Practical loop
1. When a question hinges on harness/model, read `list_runs`/`get_run` `harness`/`model`; for a report-only slug, try `get_report().providerMeta.model`, else say the report tools don't expose it and link the dashboard report page.
2. Before comparing runs or reading a trend, ask whether the combos match; flag mixed-combo deltas explicitly.
3. When triggering a run that must repeat a specific setup, pin it with `request_run`'s `harnessModel`; otherwise omit it for the default.
