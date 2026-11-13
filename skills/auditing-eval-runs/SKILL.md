---
name: auditing-eval-runs
description: >-
  Check that a set of eval runs measured the product before you report on them: shared-account collisions, text added to the task, the simulated user acting on real accounts, hidden end-goal requirements, harness asymmetry, which surface each run actually used, and whether the judge graded alike runs alike. Use before comparing lanes, harnesses or competitors across many runs, before writing findings to a company, or when results look surprising.
---

# Auditing eval runs

The per-run skills (reading-an-agent-trace, spotting-friction, interpreting-scores) assume a run measured the product. Across many runs, the setup is often the bigger cause: a shared account, a hint added to the task, a requirement the prompt never stated. Audit first, then read results. A finding the setup caused is a setup bug to fix and rerun, not a product finding.

## 1. Were the runs valid?

- **Shared account, concurrent runs.** Sort the runs by start time and group them by the credential they used. Overlapping windows on one account mean runs can see and break each other's objects. In the traces, look for objects the run didn't create: "already exists" errors, idempotency conflicts, list calls that return another run's items, a run reporting another run's result as its own. Rerun with one account per concurrent run, or space the runs out.
- **The task the agent actually got.** Read the first `user` turn verbatim and diff it against the prompt text. Anything extra (login hints, persona text, scaffold notes) is part of the task now, and the agent will follow it.
- **The simulated user's side effects.** Scan the turns where the agent asked the user. Did the simulated user paste secrets or records, create keys or webhooks, log into other services, or push a different goal? Those runs measure the simulated user, not the agent.
- **endGoal and checks vs the prompt text.** List every endGoal clause and every check, and mark the ones the prompt text never asks for. A hidden requirement fails every run, and that is a prompt bug.
- **Harness asymmetry.** Count the simulated-user replies per harness. If one harness never gets answers, any task that needs a question is unfair to it. Compare only runs that didn't need one, or say so.

## 2. Which surface did each run use?

A lane named "CLI" only measures the CLI if the agent used it. For every run, count:
- MCP tool calls
- CLI subcommands (`<bin> <subcommand>` in shell inputs)
- raw API hostnames in shell or fetch calls
- SDK imports in written code
- docs fetches

Lay it out as feature × lane × harness. A lane where most runs used a different surface measures discovery, not the surface (see evaluating-mcp-servers, "lanes name their surface"). Some harnesses call tools inside a code-execution tool, so tool histograms undercount (see harnesses-and-models).

## 3. Did the judge grade alike runs alike?

`get_report().reward` holds the verifier's per-check checklist (slug → pass/fail/null), with evidence per check. Read it, not only `outcome`. Look for:
- the same behaviour judged differently in two runs
- an `outcome` that contradicts the checklist (all checks passed but `goal_not_met`, or `succeeded` with most checks failed)
- a verdict blaming truncation when the trace shows the work finished
- default checks that can't apply to this task scoring 0 on runs that did the work, or passing for free and padding the count (see evaluating-mcp-servers on turning them off)

Report the custom-check pass rate separately from the defaults. Cite a judge disagreement as a judge finding, never as product friction.

## 4. Working in bulk

Through MCP, `get_run_trace` defaults to the last 20 turns with tool results capped. For a whole set, the `evals` CLI is faster:
- `evals runs list --prompt-id <id>`, once per prompt (and per competitor sub-prompt)
- `evals runs get-trace --run-id <id> --all true --full-tool-results true` for each run's complete trace
- render each trace to compact text with secrets redacted, then grep across runs for the patterns above

Hand older or low-priority prompts to a subagent, and keep the cross-run comparison yourself.
