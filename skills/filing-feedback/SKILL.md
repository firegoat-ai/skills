---
name: filing-feedback
description: When and how to report a platform bug, confusing behaviour, or missing feature through send_feedback.
---
# Filing feedback

## When something looks wrong
If a platform call fights you — a misleading or missing error, a response that contradicts another tool, a write that returns success but doesn't stick on a later read, input silently dropped, a field the docs promise that isn't there — call `send_feedback` (type `bug`) while it's fresh, then continue the task. Filing is a side channel, not a blocker: after filing, do what the user asked anyway with what you have, and tell them in one line what is uncertain because of the platform issue (e.g. "run requested; persona instructions may not be applied, bug filed"). Only stop if the task literally cannot proceed — the user is waiting on the outcome, and a filed bug plus a run with a caveat is more useful than a filed bug alone.

## What to include
Include the request you sent, the response you got, and what you expected; put report slugs, run/prompt ids and tool names in `context`. Do not retry the same call more than once.

## Which type
Something broken or contradictory is type `bug`. Something that works but was confusing, slow, or made you guess is type `feedback`. Something you wished existed is type `feature_request`.

## What to skip
Skip only your own mistakes and one-off transient errors.
