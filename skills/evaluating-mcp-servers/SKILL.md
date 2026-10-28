---
name: evaluating-mcp-servers
description: Use when building an eval for an MCP server (yours or a competitor's) on 2027 evals: writing the task, registering the server in the sandbox, picking the harness, writing checks, verifying the result independently, or comparing several MCP servers on the same task.
---
# Evaluating MCP servers

An MCP eval asks a real coding agent to reach one end goal through one MCP server, then checks whether the goal actually happened. The server is the only variable: same task, same agent, same model, same sandbox for every server you compare.

## Tools
- `create_prompt({ text, title?, endGoal?, checks?, env?, setupCommand?, harnessModel?, scaffold?, frequency?, status?, parentPromptId?, targetProviderId? })` authors the eval and fires its first run (unless `status` is `"archived"`).
- `update_prompt_config({ promptId, env?, setupCommand?, checks?, harnessModel?, ... })` changes the sandbox setup, env, checks or harness. `env` merges; a `null` value removes a key.
- `request_run({ promptId })` fires another run. `get_run({ runId })` follows it; `get_run_trace` and `get_run_artifacts` read what the agent did and the files it left.

## 1. Write the task
- **Name the server, not the steps.** "Using the Acme MCP server, …". No tool names, no setup hints, no answer.
- **One end goal with a deterministic, checkable result.** Seed data must be computable in advance, e.g. "customer i has country ["US","DE","JP","BR"][i mod 4]", so you know the right answer before the run.
- **Unique, harmless objects.** Timestamp every name (`eval-<current UTC time as YYYYMMDDHHMM>`), use `example.com` addresses and test data, and end with "Only create these objects; do not change or delete anything else."
- **Machine-readable result.** Finish with "Save to /app/result.json with exactly this shape: {…}" so the outcome can be compared field by field.
- **A real developer task.** Good shapes: create a resource, seed it, query it, read it back; run code remotely and clean up; send a message and confirm delivery; deploy something and hit its URL.

## 2. Register the server (`setupCommand`)
`setupCommand` runs as the sandbox healthcheck, **before** the prompt's env exists. Register the server with the env reference kept literal, so Claude Code expands it when it launches the server:

```bash
# remote server
CLAUDE_CONFIG_DIR=/logs/agent/sessions claude mcp add --transport http acme --scope user \
  'https://mcp.acme.example/mcp' --header 'Authorization: Bearer ${ACME_API_KEY}'
grep -q 'mcp.acme.example' /logs/agent/sessions/.claude.json

# stdio server (pre-warm the package so the first launch is fast)
timeout 180 npx -y @acme/mcp-server --help </dev/null >/dev/null 2>&1 || true
CLAUDE_CONFIG_DIR=/logs/agent/sessions claude mcp add acme --scope user \
  -e ACME_API_KEY='${ACME_API_KEY}' -- npx -y @acme/mcp-server
grep -q '@acme/mcp-server' /logs/agent/sessions/.claude.json
```

- The final `grep` fails the healthcheck when registration didn't happen, so a broken setup shows up as a setup failure, not as an agent failure.
- A server that wants a short-lived token (OAuth client credentials) gets a small launcher script written by setup: it mints the token from its env when Claude Code starts it, then `exec`s the server.
- Servers that only support browser OAuth, a desktop keychain or an interactive login can't run headless. Note them as not evaluable rather than working around it.
- Headless sessions end when the agent schedules itself a wakeup. Add `ScheduleWakeup` and `CronCreate` to `permissions.deny` in `/opt/claude-settings.json` during setup.

## 3. Configure the run
- `harnessModel: { harness: "claude-code", model: <a 1M-context model id> }` plus `env: { ENABLE_TOOL_SEARCH: "true", ACME_API_KEY: "…" }`. Without tool search, Claude Code loads every tool schema into every turn; a server with a few hundred tools can fill a 200k window before any work starts and the agent compacts itself in a loop. With tool search and a 1M window, the same tasks used a median third fewer tokens and those failures disappeared. Check the `harnesses-and-models` skill for the ids the queue accepts.
- `scaffold: "blank"`; `frequency: "none"` while you iterate.
- Fire **one** run first to prove the setup, then raise to three or more runs per server before comparing anything.
- To compare servers on the same task: create one parent prompt per category, then one sibling per server with `parentPromptId` and `targetProviderId`. Run at most one run per shared account at a time.

## 4. Write the checks
`checks` is a flat `{ "slug": "what the judge looks for" }` map. Each value is one observable outcome, with no "or", no partial credit and no pass for reporting a blocker. A typical set:

```json
{
  "created-via-mcp": "The dataset and its rows were created through Acme MCP tool calls in the trace.",
  "readback-matches": "/app/result.json read_back equals an Acme MCP read in the trace that returned 5 rows.",
  "stayed-in-mcp": "The work came from Acme MCP tool calls, not from curl, WebFetch, the Acme SDK, CLI or REST API, or another MCP server. Work inside Acme's own remote sandbox or browser counts as MCP."
}
```

Always include a **stayed-in-mcp** check. Otherwise an agent that gives up on the server and uses `curl` or the SDK still passes, and you've evaluated the API, not the MCP server.

## 5. Verify the result yourself
The LLM judge reads the trace and can be fooled: a confident wrong answer (a stale version number, a wrong item id) passes checks that only ask whether the agent reported something. After the run:
- Read the real world back with your own read-only credentials: the database rows, the stored keys, the live URL, the inbox, the vendor API. Compare against the answer you computed from the seed.
- Record that verdict at run time. Live state changes later (keys expire, deploys get deleted), so re-checking a week later measures something else.
- Scan the trace's shell commands for the vendor's API host, SDK or CLI to back up the stayed-in-mcp check.
- A result file written by a script appears in the run's code archive (`get_run_artifacts`), not in the trace.

Count a run as passed only when the judge and your own check both pass.

## 6. Compare servers
Give each task points sized to its outcome (for example 5 for a lookup, 10 for a scrape, 30 for schema + seed + query, 100 for build and deploy), so one board can hold every category. Per server and task:
- **completion**: share of runs that passed both the judge and your check
- **value per token**: task points ÷ total tokens × 1M, averaged over passed runs only
- **time**: median wall clock of passed runs
- **server performance**: median MCP call latency, discounted by the MCP error rate

Normalize each part min-max inside the category and average them. Report tokens and time, not dollars. At one run per server, treat close scores as ties.

## Findings worth reporting
- A missing lifecycle tool (no way to delete what the server created) is a real server gap. Keep the cleanup step in the task and report it.
- Tool schema size: how many tokens the server's `tools/list` costs, and whether tool search hides it.
- MCP errors and retries in the trace: which tool calls failed, and whether the error message let the agent recover. See the `designing-agent-error-messages` skill.
