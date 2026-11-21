---
name: using-firegoat
description: >-
  Which Firegoat surface to use and how to set it up: the MCP server and the evals CLI for a Firegoat account (evals run in the cloud), the Firegoat CLI (npx firegoat@latest) for evals on your own machine with no account, plus the REST API and dashboard. Use when someone asks how to use Firegoat, how to connect an agent to it, which CLI to install, or why a firegoat or evals command does something unexpected.
---

# Using Firegoat

Firegoat runs AX evals: a real coding agent does a task with a developer product, a judge grades the run, and you read where it got stuck. There are two ways to run them, with different tools:

| | Cloud (Firegoat account) | Local (no account) |
|---|---|---|
| Where evals run | Firegoat's sandboxes, with a browser agent for sign-ups and logins | Your machine, with your own coding agent, no sandbox |
| Tools | MCP server, `evals` CLI, REST API, dashboard | `npx firegoat@latest` (the Firegoat CLI) |
| Tests live in | Your org's prompts in Firegoat | `firegoat.yaml` in your repo |
| Graded by | A judge: end goal plus checks, with metrics and insights | Script checks set the exit code; your coding agent judges the end goal itself |
| Start with | Request access at https://firegoat.ai/request-access, then a trial | `npx firegoat@latest init` |

The two CLIs are different programs. `evals` talks to your Firegoat account; `npx firegoat@latest` runs evals locally. Commands from one don't exist in the other.

## Cloud: the MCP server
The MCP server gives an agent the same read and write access as the dashboard: prompts, runs, reports, traces, briefs, competitors, feedback. It is a remote HTTP server at `https://firegoat.ai/evals/api/mcp`.

- **Claude Code, the Firegoat plugin:** `/plugin install firegoat --marketplace firegoat-ai/skills` adds the server (as `plugin:firegoat:firegoat`), the `evals` CLI and these skills. Then run `/mcp`, pick the firegoat server and sign in with your Firegoat account in the browser.
- **Claude Code, by hand:** `claude mcp add --transport http --scope user firegoat https://firegoat.ai/evals/api/mcp`, then the first tool call (or `/mcp`) opens the sign-in.
- **Codex:** `codex mcp add firegoat --url https://firegoat.ai/evals/api/mcp`, then `codex mcp login firegoat`.
- **Cursor, VS Code, Claude Desktop:** the same URL; see https://firegoat.ai/evals/docs/mcp.
- **CI or headless, no browser:** send an API key as `Authorization: Bearer <key>`. Create the key under API keys in your org's settings.

The agent acts as the signed-in user, in the org of their email domain. Everything is org-scoped: you only ever see your own org's data. To switch accounts, remove the server and add it again.

## Cloud: the evals CLI
`evals` is the same API from a terminal, scripts and CI. It needs a Firegoat account.

```sh
curl -fsSL https://firegoat.ai/evals/install.sh | sh   # installs ~/.local/bin/evals (macOS, Linux x64)
evals auth login                                       # browser + device code
evals whoami                                           # account and org it acts as
evals prompts list
evals prompts create --title "<name>" --text "<prompt text>" --end-goal "<what success looks like>"
evals runs request --prompt-id <prompt-id>
evals runs get --run-id <run-id>
```

- In CI, export `EVALS_TOKEN` (the CLI reads `EVALS_TOKEN`, not `EVALS_API_KEY`). It wins over a stored login.
- `--format json`, `--query <JMESPath>`, `--dry-run`, and `--schema` on any command (its flags as JSON, for agents).
- If `evals` isn't found after install, `~/.local/bin` isn't on `PATH`; the installer prints the line to add.
- With the Firegoat plugin, `evals` is already on Claude's PATH: the first call installs the official release into `~/.cache/firegoat-plugin/bin`. An `evals` you installed yourself takes precedence.

Pick the MCP server when an agent is working interactively, and the CLI for scripts, CI and shells without MCP. Both reach the same data; the REST API under `https://firegoat.ai/evals/api/v1` is the third door (spec at `https://firegoat.ai/evals/api/openapi`).

## Local: the Firegoat CLI
`npx firegoat@latest` runs evals in your repo with your own coding agent (Claude Code by default, `harness: codex-cli` or `grok-cli` in `firegoat.yaml`). No account needed. It needs Node 20+ and the coding agent installed and logged in.

```sh
npx firegoat@latest init     # firegoat.yaml + .firegoat/, skills, a prompt for your agent
npx firegoat@latest eval     # runs every test, checks set the exit code
npx firegoat@latest web      # browse runs in a local page
npx firegoat@latest login    # optional: upload each run's session to firegoat.ai
```

- Always use `@latest`: plain `npx firegoat` can keep running an old cached copy.
- Each run gets a fresh, empty folder and a fresh agent config (none of your settings, skills or MCP servers), so the agent starts cold like your users' agents. There is no sandbox: it runs as you.
- Exit codes: 0 all checks passed, 1 a check failed, 2 a run wasn't graded (agent missing or logged out, crash, timeout, build failed), 3 usage or config error.
- To write the tests, use the firegoat-onboarding skill.

## Which skill next
- Reading results: interpreting-scores, reading-an-agent-trace, spotting-friction.
- Writing cloud prompts: suggesting-eval-prompts, evaluating-mcp-servers, harnesses-and-models.
- Comparing: comparing-competitors, placing-on-the-ax-tier-list.
- Sharing: linking-to-the-dashboard, writing-briefs.
- Something in Firegoat itself looks broken: filing-feedback.

Docs: https://firegoat.ai/evals/docs (and https://firegoat.ai/llms.txt for agents).
