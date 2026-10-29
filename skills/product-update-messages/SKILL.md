---
name: product-update-messages
description: Use when writing a product update about what Firegoat shipped for a customer's shared Slack channel (or email), so every shipped feature is linked to where it lives in the product and the update ends with concrete offers to use it.
---
# Writing product updates

The reader is a team that already uses Firegoat evals. They want to know what they can do now that they couldn't last month, and where to click to try it. Write from the founder's voice: warm, short, specific.

## Template
```
Hi <team>, quick update on what we shipped in <month>: <one-line theme, e.g. "you can now see the route agents take through your product">.

*<Feature name>*
<What it lets them do, in one or two sentences.> <Where it lives: "Every prompt has a new <Paths tab|link>".> <Optional: why we built it, e.g. "Many of you asked for this".>

*<Feature name>*
...

*Coming soon*
• <item>
• <item>

Want me to <offer tied to feature 1>? Or <offer tied to feature 2>?
```

## Link everything that shipped
Every shipped feature gets a link to the exact place it lives in the product, not to the dashboard home. "Paths tab" links to that prompt's Paths tab, "Settings" links to Settings, "install the MCP" links to the install page. If the reader would have to hunt for it, the link is wrong.
- Link the words that name the feature or the click path ("Prompt → Configure"), not a bare URL on its own line.
- When the update is for one team's channel, link into their own data: their prompt, their run, their Insights tab (see the linking-to-the-dashboard skill).
- Posts, talks and events get links too (blog post, recording, event page).
- In Slack, use `<url|text>` links. Never paste the same URL twice.
- If a feature has no page to link to yet, say how to get it instead ("reply here and we'll turn it on").

## Each feature
- Bold name, then what the reader can do with it, not how we built it. "Run any prompt across 28 agent and model combinations" beats "added harness support".
- Say where it is in the product (tab, page, button) and link it.
- Use real numbers when we have them, with the comparison: "28 combinations, up from 20 in August".
- Credit the ask when it came from customers: "Many of you asked for it", "You raised variance between runs: we heard you".
- Up to 5 features, most useful first. Group small related changes under one heading (e.g. Scenarios, Checks and persona instructions under "Prompt flexibility").

## Beyond features (optional)
- *Coming soon*: 2 or 3 bullets, each one line. Only things we're actually building.
- *More from us*: a blog post, talk, or event, each linked. One line each.

## Close with offers, not just "thoughts?"
End with one or two specific offers based on what shipped, phrased as questions the bot can act on if they say yes:
- New models shipped → "Want me to rerun your getting-started prompt on Opus 5.5 and Fable 5.1?"
- Competitor benchmarking shipped → "Want me to add <competitor> to your <prompt> so you can see where you lead?"
- Scenarios or Checks shipped → "Want me to draft a scenario for your CLI path, or suggest checks for <prompt>?"
- An integration shipped (GitHub app, MCP, Slack) → "Want me to send the install link to whoever owns your docs repo?"

Tailor the offers to the channel's team when you have their data (their prompts, competitors, recent runs). Then one line inviting replies: "Anything we should build next? Reply here."

## Voice
- Open with one sentence, then go straight to features. No long intro about what Firegoat is.
- Plain words, short paragraphs, a few emoji at most.
- Always "Firegoat", never "2027" or "2027.dev". No em dashes; use a colon, comma or new sentence.
- Slack: no merge tags, no signature block, no unsubscribe footer. Under 250 words for the core update.

## Before sending
- Every shipped feature, post and event is linked to its exact page.
- Each feature says what the reader can do and where to click.
- Numbers are real and have a comparison point.
- Up to 5 features, most useful first.
- Ends with one or two specific offers tied to what shipped.
- No em dashes, no "2027".
