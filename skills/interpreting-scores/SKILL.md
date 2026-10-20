---
name: interpreting-scores
description: Use when reading or explaining an eval run's metrics, insights, or overall experience, or when a run has no score.
---
# Interpreting a run

The weighted `score`/`grade` and the per-dimension breakdown have been REMOVED from the API output — `get_report()` no longer returns `score`, `grade`, `dimensions`, `aiDiscoverability`, `scoringFormula`, or `weightedCalculation`. Do not ask for them and do not invent them. Explain a run from its hard metrics and its insights, never from a letter grade or an aggregate number.

## Metrics are the primary signal
`get_report().metrics` carries the concrete, measured facts: time-to-complete, cost, error count, interruption count, tool-call count/breakdown, and similar. These are legible on their own — "the agent made 14 tool calls and hit 3 errors before finding the API key" explains more than any grade ever did. Lead with these numbers, and be honest about the missing ones: a metric that is null is "not measured", never a zero.

## Insights are the other half
The judged, human-readable takeaways ride the report as `summary`, `recommendations`, and `legacy.keyFinding` (`get_report()`), plus the org's insight board (`list_insights`). Use these for the "what happened and what to do about it" narrative — the friction the agent hit, what went well, and the concrete fixes. Pair them with the metrics that back them up.

## A run with no score is NOT a failure
Runs still carry an `outcome` even though no score is exposed. `get_report().outcome` is the enum: `succeeded` (the agent completed a gradeable session), or one of the nulled causes — `goal_not_met`, `no_creds`, `excluded` (manually nulled), or `scoring_failed`. `get_report().nullNotice` is the one-line cause string the queue/admin prepended (null when `outcome === "succeeded"`). A non-`succeeded` outcome means "not scored / incomplete", not a zero — don't treat it as a bad result and don't average it into trends. If a user asks *why* a run did not complete, cite `outcome`/`nullNotice` first; reach for `get_trace` only to show the moment things went wrong, not to recover the reason code.

## Practical loop
1. `get_report` → read `metrics` (time, cost, errors, interruptions, tool calls) and the insight fields (`summary`, `recommendations`, `legacy.keyFinding`).
2. Check `outcome`; if it is not `succeeded`, cite `outcome` (and `.nullNotice`) for the cause, and inspect `get_trace` only if the user wants the moment things went wrong.
3. To explain a run, lead with the metrics that stand out plus the key finding and recommendations, then map them to friction (see the spotting-friction skill). Never cite `score`/`grade`/`dimensions` — they are gone from the API.
