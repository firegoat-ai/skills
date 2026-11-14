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

## How the two agents talk
The code agent sees the browser agent's messages as messages from the user. When the code agent asks for a key or a login, the browser agent's answer arrives in the code agent's trace as a user turn. Read those turns as the browser agent speaking, not as a human or as the prompt, and match them to the screenshot turns around the same timestamp.

## Reading a browser segment
- `screenshotPageUrl` tells you which page the agent was on — a login form, an OAuth consent screen, a signup flow, dashboard settings.
- A run of screenshots on the **same URL** does not by itself mean the browser agent is stuck. Single-page apps and embedded login widgets (Descope, Clerk, Auth0) change the screen without changing the URL, so a whole signup or login flow can sit on one URL. `screenshotPageUrl` is captured at the same moment as the screenshot, so a run of identical URLs is real, not a logging artifact.
- Before calling a same-URL run stuck, read each step's `screenshotAction` and `screenshotThinking`, and look at the images on the dashboard. Progress shows as different actions on different fields (email, then code, then a consent button). Stuck shows as the same action repeated, or thinking that says the page didn't change: a signup wall, a captcha, a login loop.
- `screenshotThinking` narrates the confusion in the agent's own words — quote it.
- This is where browser-side friction lives (auth walls, missing credentials, OAuth flows the agent can't drive), as opposed to code-agent friction (bad docs, broken SDKs) covered by the spotting-friction skill.

## Practical loop
1. `get_report` for context.
2. `get_trace` with `includeBrowser: true` — this also makes turn indices match the dashboard's `#turn-N` anchors (see the linking-to-the-dashboard skill).
3. Scan the screenshot turns for repeated actions and stuck `thinking`. A same-URL run is only a loop if the actions repeat too.
4. When reporting browser-side friction, quote the `screenshotPageUrl` + `screenshotAction`/`screenshotThinking` as evidence, and link the dashboard trace turn so the human can see the actual image you can't.
