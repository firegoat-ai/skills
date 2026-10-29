---
name: product-update-messages
description: Use when writing a product update about what Firegoat shipped, posted by the Slack bot to customer channels (or sent as an email), so every shipped feature is a linked one-liner and the update ends with offers the bot can act on.
---
# Writing product updates

The reader is a team that already uses Firegoat evals. They want to know what they can do now that they couldn't before, and where to click to try it. One general update goes to every channel; don't personalize it per team.

## Slack format
One message, no thread. About 120 words.
```
<One line: the most useful thing we shipped, and what it lets them do.>

• *<url|Feature name>*: <what they can do with it, one line>
• *<url|Feature name>*: <one line>
• *<url|Feature name>*: <one line>

<One or two offers the bot can act on.>
```

- The first line is what people see in the channel before opening it. Lead with the most useful feature, not "Hi team, quick update".
- Up to 5 features, most useful first, each one line.
- No Coming soon, blog posts or events. Only what shipped.
- Post with `unfurl_links: false` so a list of links doesn't turn into a wall of preview cards.
- Never use @channel or @here.
- If little shipped this cycle, skip the update rather than post a thin one.

## Each feature is a linked one-liner
- Link the feature name to the exact place it lives in the product, not the dashboard home: "Paths tab" links to the Paths tab, "install the MCP" links to the install page. Use Slack's `<url|text>` format, no bare URLs.
- Say what the reader can do with it, not how we built it. "Run any prompt across 28 agent and model combinations" beats "added harness support".
- Use real numbers when we have them, with the comparison: "28 combinations, up from 20 in August".
- Group small related changes into one bullet (e.g. Scenarios, Checks and persona instructions as "Prompt flexibility").
- If a feature has no page to link to yet, say how to get it ("reply here and we'll turn it on").

## End with offers the bot can act on
Close with one or two offers tied to what shipped, phrased so a reply triggers the action:
- New models → "Reply *rerun* and I'll run your prompts on Opus 5.5 and Fable 5.1."
- Competitor benchmarking → "Want me to add a competitor to one of your prompts? Reply with the name."
- Scenarios or Checks → "Reply *checks* and I'll suggest checks for your prompts."
- An integration (GitHub app, MCP) → "Want the install link for whoever owns your docs repo?"

Only offer what the bot can actually do from a reply.

## Voice
- Speak as the Firegoat bot, in first person ("I'll rerun..."). No sign-off, no founder name, no signature.
- Plain words, a few emoji at most.
- Always "Firegoat", never "2027" or "2027.dev". No em dashes; use a colon, comma or new sentence.

## Email version
The monthly email can carry the same features at full length: a greeting, a short section per feature with its link and a button, and a "Thoughts? Reply directly" close. Same linking and voice rules, except the email is signed by Mika.

## Before posting
- One message, about 120 words, leading with the most useful feature.
- Every feature is a one-line bullet linked to its exact page.
- Numbers are real and have a comparison point.
- Ends with one or two offers the bot can act on from a reply.
- Speaks as the bot, link previews off, no @channel, no em dashes, no "2027".
