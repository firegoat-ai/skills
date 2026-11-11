---
name: product-eval-methodology
description: >-
  How to evaluate a developer product's agent experience (AX) end to end on Firegoat evals, from finding its agent-facing surfaces to reading the first results. Use when asked to "evaluate <product>", "set up evals for <company>", or when onboarding a product's CLI, MCP server, API, SDK or docs into Firegoat evals.
---

# Product eval methodology

Ten steps, in order. Tools: the Firegoat evals MCP server (`create_prompt`, `update_prompt_config`, `add_credential`, `request_run`) or the `evals` CLI over the same API.

1. **Discover the surfaces.** From a URL or CLI name, list each agent interface on one numbered line:
   - CLI: the install line (`npx …` / `brew install …` / curl binary URL)
   - MCP: server name + URL
   - API: OpenAPI link, if found
   - SDK: package name
   - Docs: link

2. **User picks.** The user names the surfaces to evaluate by number ("1, 2, 5"). Do not pick for them.

3. **Who are the users?** The user picks agent categories:
   - coding agents (Claude Code, Codex, OpenCode…)
   - coding apps (v0, Lovable, ChatGPT sites)
   - chat assistants (Claude, ChatGPT)
   - proactive personal assistants (Poke, Grok bot…)
   - browser agents (Claude in Chrome, Vercel agent-browser)

   The category sets the capability: browser + CLI, CLI only, or MCP only. That sets the runner (`harnessModel`); see the harnesses-and-models skill.

4. **Setup: install + auth, one step.**
   - Install: the prompt's `setupCommand` pre-installs the CLI / MCP / SDK version under test. Docs and OpenAPI need no install.
   - Auth: ask the user for an API key. Store it with `add_credential` (sensitive). Inject it into the prompt env with `envCredentials`, so the CLI / MCP / SDK starts logged in.
   - Some prompts skip auth on purpose: "can the agent figure it out alone".

5. **Local vs cloud.** If the product has an OSS/local and a cloud offering, measure both. Focus on cloud.

6. **Find the features.** From the docs/quickstart, take the ~5 core features the company makes money on. Keep core platform features (DB, auth) apart from secondary ones.

7. **Pick the starting state.**
   - Core features: cold start (`scaffold`: `blank`, or a bare `nextjs` / `react`).
   - Deeper/secondary features: an existing app that already integrates the product (e.g. create-next-app with the product wired in), built by `setupCommand`.

8. **Write the prompt.** Reverse-engineer the business problem. 1–2 lines in a real user's words. No hints, no product names, no implementation names. One exception: when you compare surfaces (CLI vs MCP vs SDK lanes), each lane names its surface, or the lane measures whether the agent found it rather than how well it works. Keep an unnamed baseline separately if you want that discovery signal. See the suggesting-eval-prompts skill.

9. **Write the checks.** The knowledge lives here, not in the prompt. The `checks` map (slug → criterion) says what a correct implementation is and how the agent was supposed to use the product (right command, MCP tool, endpoint, SDK call). `endGoal` is the outcome the judge grades.

10. **Run + read the results.** `create_prompt` fires the first run; `request_run` runs it again. Follow it with `list_runs` / `get_run`, then read the finished run with `get_report` (see the interpreting-scores and spotting-friction skills) and give the user a dashboard link to it (linking-to-the-dashboard).
