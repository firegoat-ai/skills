---
name: reading-browser-screenshots
description: Use when a session trace contains browser-agent screenshot turns, or when a run's friction happened in the browser (login, signup, OAuth, dashboard) rather than in the code agent's terminal.
---
# Reading browser screenshots

Every eval run pairs a code agent (the terminal, e.g. claude-code) with a **browser agent** (browser-use based) that handles the human-ish web work: logins, signups, OAuth flows, grabbing API keys from dashboards. When friction happened in a browser — an auth wall, a signup form, a dashboard the agent couldn't navigate — the evidence lives in the browser agent's screenshot turns, not the code agent's tool calls.

## Tools
- `get_trace({ slug, includeBrowser: true })` — interleaves the browser agent's screenshots into the turn list, timestamp-ordered, as turns of `type: "screenshot"`. Screenshots lacking real timestamps are stacked after the last code-agent turn.
- `get_report({ slug })` — for the surrounding context (the prompt, `metrics`, `legacy.keyFinding`) before diving in.

## What a screenshot turn carries
Each screenshot block has:
- `screenshotStep` — the browser agent's step number.
- `screenshotThinking` — the browser agent's reasoning at that moment, in its own words.
- `screenshotAction` — what it did (e.g. a navigate, a click).
- `screenshotPageUrl` — the URL of the page it was on.
- a `timestamp`.

## You read text, not pixels
The chat/MCP tools return **no image bytes** — each screenshot block carries a `screenshotUrl`, but it is a relative `/api/screenshot` proxy path that only resolves on the authenticated dashboard, not an image you can fetch or see. You analyze the browser agent entirely through its `thinking`/`action`/`pageUrl` text. The actual images are only viewable on the authenticated dashboard trace page, so when it matters, link the human there (see the linking-to-the-dashboard skill).

## Reading a browser segment
- `screenshotPageUrl` tells you which page the agent was on — a login form, an OAuth consent screen, a signup flow, dashboard settings.
- A long run of screenshots on the **same URL** means the browser agent is stuck: a signup wall, a captcha, a login loop.
- `screenshotThinking` narrates the confusion in the agent's own words — quote it.
- This is where browser-side friction lives (auth walls, missing credentials, OAuth flows the agent can't drive), as opposed to code-agent friction (bad docs, broken SDKs) covered by the spotting-friction skill.

## Practical loop
1. `get_report` for context.
2. `get_trace` with `includeBrowser: true` — this also makes turn indices match the dashboard's `#turn-N` anchors (see the linking-to-the-dashboard skill).
3. Scan the screenshot turns for page-URL loops and stuck `thinking`.
4. When reporting browser-side friction, quote the `screenshotPageUrl` + `screenshotAction`/`screenshotThinking` as evidence, and link the dashboard trace turn so the human can see the actual image you can't.
