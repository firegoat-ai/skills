---
name: reading-an-agent-trace
description: Use when you need to reconstruct what an agent actually did during an eval run, or localize where it got stuck, from the session log.
---
# Reading an agent trace

An eval run records the full agent session as a turn-by-turn trace. Read it with `get_trace`; pull run-level context first with `get_report` so you know the metrics, the key finding, the prompt, and which provider was under test.

## Tools
- `get_trace({ slug, offset?, limit?, includeBrowser? })` — the parsed conversation log for a report.
  - Default returns the **last 20 turns** (`offset` defaults to `totalTurns - limit`). To read from the start, pass `offset: 0`. Page forward with `offset`/`limit`.
  - `includeBrowser: true` interleaves the browser-agent screenshots (with each shot's thinking/action/page URL) in timestamp order. Omit it when you only care about the code agent (see the reading-browser-screenshots skill for how to analyze them, and note the dashboard's `#turn-N` anchors assume screenshots are merged in).
  - Returns: `{ slug, totalTurns, showing:{offset,limit,count}, totalToolCalls, totalErrors, totalInterruptions, toolCounts, turns[], setupCommand, healthcheckLog }`. `toolCounts` is a per-tool call histogram — scan it first to see where the agent spent its effort.
  - `setupCommand` is the bash script that ran before the agent, and `healthcheckLog` is that script's own output. When a run failed early or the agent started from a broken environment, read the log first — the cause is usually there, not in the turns.
- `get_report({ slug })` — for the surrounding context (metrics, legacy.keyFinding, summary, recommendations, the prompt).

## The turn model
Each turn has `type`, `content[]` (ContentBlock array), `hasError`, `toolNames[]`, and timing (`timestamp`, `durationMs` or `gapAfterMs`).

Turn types:
- `user` — the initial prompt (first user turn) and genuine trailing user notes.
- `assistant` — the agent's thinking/text/tool calls.
- `interruption` — a user message injected mid-run, or the user's answer to an `AskUserQuestion` call. `totalInterruptions` counts both; a high count means the agent needed hand-holding.
- `system` — a `turn_duration` divider; its `durationMs` is how long that step took.
- `screenshot` — only present when `includeBrowser` is set.

ContentBlock `type` values inside a turn: `text`, `thinking`, `tool_use`, `command`, `screenshot`, `question_answer`.

## Questions the agent asked
`AskUserQuestion` is a normal `tool_use` block in an `assistant` turn; its `toolInput` is the question(s) as the agent asked them. The user's answer is an `interruption` turn whose content is a `question_answer` block: `questions[]`, each with `question`, `options[]` (`{label, selected}`, ending with a "Type something" option that is selected when the answer was not one of the offered options) and `typed` (the user's free-text answer when "Type something" is selected, else null). Answers given together share one interruption turn. An unanswered or failed question has no answer turn.

## Localizing where things went wrong
A `tool_use` block is **merged with its `tool_result`** by `tool_use_id`. When a tool result came back with an error, that block has `isError: true` and its assistant turn has `hasError: true`. Use this to separate two very different failures:
- **Tool failed** — `isError` is set on a `tool_use` block: the tool (a curl, an SDK call, a CLI command) returned an error. This is usually a signal about the *provider's product* (a 404, a bad SDK slug, a confusing error message the agent then had to recover from).
- **Agent got stuck** — no tool error, but the agent loops, re-reads the same docs, retries the same call, or the user has to interrupt. Look for repeated `toolNames`, rising `gapAfterMs`, or an `interruption` turn right after. This is an agent-experience friction signal even when nothing "errored".

## Timing
If the log carries `turn_duration` system turns, per-step time is `durationMs` on those turns. Otherwise each turn gets a `gapAfterMs` = wall-clock gap to the next turn. Large gaps flag slow tool calls or the agent stalling.

## Practical loop
1. `get_report` for the prompt + `metrics` (`toolCalls`, `errors`, `interruptions`, `time`) + `legacy.keyFinding`.
2. `get_trace` with `offset: 0` for the opening (how the agent started), then scan `toolCounts`/`totalErrors`.
3. Jump to the errored turns and interruptions to pinpoint the moment. Page with `offset`/`limit` for long runs — don't assume the default last-20 window contains the failure.
4. Report the specific turn(s) and whether it was a tool failure vs the agent stalling, quoting the tool name and the error text.
