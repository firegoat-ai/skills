# Firegoat Skills

Open-source [Claude Code](https://claude.com/claude-code) skills for building software that **AI agents** can actually use — by the team behind [Firegoat](https://firegoat.ai), where we run real agents through 100+ dev tools and score their experience.

Each skill is a self-contained folder under [`skills/`](./skills). This repo is the single source for every Firegoat skill, including the product skills the [Firegoat evals](https://firegoat.ai/evals) dashboard installs.

## Skills

### Building for agents

| Skill | What it does |
|---|---|
| [`designing-agent-error-messages`](./skills/designing-agent-error-messages) | Audit and rewrite your error messages, exceptions, and API/SDK/CLI failures so an agent can self-correct from the message alone. Grounded in patterns from 3,500+ agent runs across 136 dev tools. |
| [`agent-auth`](./skills/agent-auth) | Log a real human into a dev tool's CLI by auto-opening the OAuth browser popup and letting the login complete itself — no token copy-paste, no browser automation. Sanity + Netlify recipes plus a verified login reference for 19 dev-tool CLIs. |
| [`vent-friction`](./skills/vent-friction) | When your agent hits real friction with a third-party dev tool — a bug, broken docs, a misleading error — it captures the repro while it's fresh and files a dense, reproducible issue to the tool's makers via `npx agentspa vent`, only after you say yes. |

_More coming — discoverability, agent-friendly docs, SDK design._

### Firegoat evals product skills

These teach your agent to work with your [Firegoat evals](https://firegoat.ai/evals) data through the Firegoat evals MCP server: reading scores, traces, and friction.

| Skill | What it does |
|---|---|
| [`interpreting-scores`](./skills/interpreting-scores) | Read and explain an eval run's metrics, insights, and overall experience, including runs with no score. |
| [`reading-an-agent-trace`](./skills/reading-an-agent-trace) | Reconstruct what an agent did during an eval run and find where it got stuck. |
| [`reading-browser-screenshots`](./skills/reading-browser-screenshots) | Read browser-agent screenshot turns when friction happened in login, signup, OAuth, or a dashboard. |
| [`spotting-friction`](./skills/spotting-friction) | Find what hurt an agent's experience and turn it into a fix the tool owner can ship. |
| [`comparing-competitors`](./skills/comparing-competitors) | Benchmark a provider against its competitors on the same eval prompt. |
| [`harnesses-and-models`](./skills/harnesses-and-models) | Work out which coding agent and model ran an eval, compare across them, or pin one for a run. |
| [`suggesting-eval-prompts`](./skills/suggesting-eval-prompts) | Write eval prompts that measure the agent experience of a real end goal. |
| [`evaluating-mcp-servers`](./skills/evaluating-mcp-servers) | Build an eval for an MCP server: register it in the sandbox, write the task and checks, verify the result yourself, and compare servers head to head. |
| [`team-findings-messages`](./skills/team-findings-messages) | Write eval findings as a short message to the evaluated company's engineers: the setup, the most important verified findings with linked trace turns, a fix their team controls for each, and a next step. |
| [`linking-to-the-dashboard`](./skills/linking-to-the-dashboard) | Cite runs, prompts, and trace moments as clickable dashboard links. |
| [`filing-feedback`](./skills/filing-feedback) | Report a platform bug, confusing behaviour, or missing feature through `send_feedback`. |

### firegoat

For [`npx firegoat`](https://www.npmjs.com/package/firegoat): local AX evals that run a coding agent against your SDK, CLI or docs from your own repo.

| Skill | What it does |
|---|---|
| [`firegoat-onboarding`](./skills/firegoat-onboarding) | Draft 2-3 `firegoat.yaml` tests for the current repo: prompts, goals, script checks, and `build:` / `install:` for the local build. Self-contained, prompt-writing principles included. `npx firegoat@latest init` installs it. |

## Install

Install every skill with the [`skills`](https://www.npmjs.com/package/skills) CLI:
```bash
npx skills add firegoat-ai/skills
```

Pick specific skills with `--skill`, and add `-g` to install them for all your projects instead of only this one:
```bash
npx skills add firegoat-ai/skills -g --skill agent-auth --skill vent-friction
```

Without the CLI, copy a skill folder into `.claude/skills/` (this project) or `~/.claude/skills/` (all projects):
```bash
git clone https://github.com/firegoat-ai/skills.git /tmp/skills
mkdir -p ~/.claude/skills
cp -r /tmp/skills/skills/agent-auth ~/.claude/skills/
```

Then in Claude Code each skill loads automatically when it's relevant — or invoke one directly: `/agent-auth`, `/interpreting-scores`.

## See where a real agent gets stuck in your product

These skills fix what you can grep for. To see where a real agent actually gets stranded in *your* onboarding — and how you rank against 135 other dev tools — check out the **[agent arena](https://firegoat.ai/arena)** and **submit a tool eval request**.

## License

MIT © [Firegoat](https://firegoat.ai)
