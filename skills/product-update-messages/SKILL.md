---
name: product-update-messages
description: Use when writing a product update about what Firegoat shipped, posted by the Slack bot to customer channels (or sent as an email), so every shipped feature is linked to where it lives in the product and the update ends with offers the bot can act on.
---
# Writing product updates

The reader is a team that already uses Firegoat evals. They want to know what they can do now that they couldn't last month, and where to click to try it. One general update goes to every channel; don't personalize it per team.

## Slack format
Post a short main message, then put everything else in a thread reply under it.

Main post (about 120 words):
```
<One line: the most useful thing we shipped, and what it lets them do.>

• *<url|Feature name>*: <what they can do with it, one line>
• *<url|Feature name>*: <one line>
• *<url|Feature name>*: <one line>

<One or two offers the bot can act on.> More in the thread 🧵
```

Thread reply:
```
• *<url|Feature name>*: <one line>   (features 4 and 5, if any)

*Coming soon*
• <item>

*More from us*
• <url|Blog post, talk or event>: <one line>

Anything we should build next? Reply here. Mika and the Firegoat team
```

- The first line is what people see in the channel before opening it. Lead with the most useful feature, not "Hi team, quick update".
- Up to 3 features in the main post, up to 5 in total, most useful first.
- Post with `unfurl_links: false` so a list of links doesn't turn into a wall of preview cards.
- Never use @channel or @here.
- If little shipped this cycle, skip the update rather than post a thin one.

## Link everything that shipped
Every shipped feature gets a link to the exact place it lives in the product, not to the dashboard home. "Paths tab" links to the Paths tab, "Settings" links to Settings, "install the MCP" links to the install page. If the reader would have to hunt for it, the link is wrong.
- Link the feature name or the click path ("Prompt → Configure"), using Slack's `<url|text>` format. No bare URLs.
- Posts, talks and events get links too.
- If a feature has no page to link to yet, say how to get it ("reply here and we'll turn it on").

## Each feature
- Say what the reader can do with it, not how we built it. "Run any prompt across 28 agent and model combinations" beats "added harness support".
- Use real numbers when we have them, with the comparison: "28 combinations, up from 20 in August".
- Credit the ask when it came from customers: "Many of you asked for it".
- Group small related changes under one bullet (e.g. Scenarios, Checks and persona instructions as "Prompt flexibility").

## Offers the bot can act on
End the main post with one or two offers tied to what shipped, phrased so a reply triggers the action:
- New models → "Reply *rerun* and I'll run your prompts on Opus 5.5 and Fable 5.1."
- Competitor benchmarking → "Want me to add a competitor to one of your prompts? Reply with the name."
- Scenarios or Checks → "Reply *checks* and I'll suggest checks for your prompts."
- An integration (GitHub app, MCP) → "Want the install link for whoever owns your docs repo?"

Only offer what the bot can actually do from a reply.

## Voice
- Plain words, one line per item, a few emoji at most.
- The bot speaks for the team. No founder signature block, no "set up a shared Slack" invite.
- Always "Firegoat", never "2027" or "2027.dev". No em dashes; use a colon, comma or new sentence.

## Email version
The monthly email can carry the same content at full length: a greeting, a short section per feature with its link and a button, Coming soon, More from us, and a "Thoughts? Reply directly" close signed by Mika. Same linking and voice rules.

## Before posting
- Main post leads with the most useful feature, has 3 features or fewer, and is about 120 words.
- Every shipped feature, post and event is linked to its exact page.
- Numbers are real and have a comparison point.
- One or two offers the bot can act on from a reply.
- Link previews off, no @channel, no em dashes, no "2027".
