---
name: placing-on-the-ax-tier-list
description: >-
  How to place a developer product on the AX tier list (S hands-off, A just confirm, B copy a key, C figure it out, D blocked): the four criteria, the grading rules, how to measure it without hinting the answer, and the discovery column. Use when grading how far an agent gets with a product before it needs a human, giving a run or provider its AX tier list placement, writing an onboarding eval to measure it, or explaining a placement on firegoat.ai/axlist.
---

# Placing a product on the AX tier list

The [AX tier list](https://firegoat.ai/axlist) grades a provider by how much of a human an agent needs before it can start using the service. It grades the provider's onboarding, not the agent: an agent's blind spot is a discovery result, not a lower tier.

## The ladder

| Tier | Label | What the user has to do | Login flow | Bot protection | Paywall | Sign up |
|---|---|---|---|---|---|---|
| **S** | Hands-off | Nothing. The agent starts using the service on its own | Anonymous or claimable resource, agent-auth (auth.md, x402) | None | No card. Free tier, or the agent pays itself (x402) | Not needed |
| **A** | Just confirm | Only confirms actions in a straightforward flow | CLI login (device code, OAuth confirm) | None, or agent-solvable | No card | Google or GitHub, one click |
| **B** | Copy a key | The user, or their browser agent, does one simple action on the website | One key, copied from a dashboard | Sign up only, one captcha | Card on file | Email + OTP |
| **C** | Figure it out | The user, or their browser agent, has to think, read how the integration works, or take several complicated actions | 2+ keys, multi-step, keys hard to find | Sign in + sign up | Card required to start | Email + verify domain, or approval |
| **D** | Blocked | Can't start | No self-serve path | Blocks everything | Paid only, sales call | Invite, waitlist, KYC |

D is the gate that stops a start outright: payment, a sales call, an invite. A dashboard that a browser agent struggles with is not D. That shows up as interruptions in the run, not as a provider tier.

## Grading rules

1. **The tier is the worst criterion.** Grade login flow, bot protection, paywall and sign up separately. When they disagree, the provider gets the worst of the four.
2. **Judge every criterion on the path that set the tier.** If the best path (say, an anonymous deploy) needs no sign-up, a captcha on a worse alternate sign-up form doesn't count against it. Mention the worse path in the comment.
3. **Paywall asks whether a human must enter a card or take a sales call**, not whether the product is free. Pay-per-call that the agent settles itself (an x402 gateway) is S.
4. **Cloud only.** Grade the hosted product. Self-hosted and local paths are out of scope, even when they need no account.
5. **A run overrides the docs only where the agent attempted that path.** If no run tried the documented no-account path, the docs stand, and the gap goes in the discovery column.
6. **"Broken" means the provider's path broke, not your harness.** If the run stopped on something in your own sandbox (no funded wallet, a missing key for another service in your env), that is not the provider's failure. Re-grade from what the provider's path did.
7. **Never publish a label its own evidence contradicts.** If the label says a path was refused and the cited docs say it works, fix one of them first.
8. **One verified path can settle several cells.** If you ran the provider's no-account claim command yourself and it provisioned a working resource, sign up, bot protection and paywall follow from it, at high confidence.

## Evidence

Tag each criterion with where its grade comes from:
- **Trace**: an eval run's session shows it. Link the turn (see linking-to-the-dashboard).
- **Docs**: the provider's documentation says so. Link the page.
- **Checked**: a command you ran yourself and saw work.

Look at real traces before deciding, not at the flow you imagine. Confidence is high when the tier rests on a trace or a check, and lower when it rests on docs alone.

## Discovery

Discovery is a fifth column that records how much of the tier the agent actually reached. It is diagnostic and never moves the tier.

| Value | Meaning |
|---|---|
| Used | The agent took the best path the provider offers. |
| Told | It took the best path, but only after the prompt or scaffold pointed at it. |
| Broken | It found the better path, and the provider's path broke on the way. |
| Missed | A documented path would have scored a better tier, and the agent never tried it. |
| None | No better path exists. Nothing to miss. |
| Not needed | The task never required a credential, so there was no better path to find. |
| No coverage | No run exercised the provider's onboarding. |

Use Missed only when the path the agent missed would have been a better tier than the one it took. A missed path at the same tier goes in the comment.

## Measuring it

- Write a one-line custom prompt for each provider about the simplest real problem that needs the service. Don't hint at the right path: no "use the claim command", no "sign up anonymously". Otherwise the result is Told, not Used. See suggesting-eval-prompts.
- In a run, "asked the user" means the browser agent stood in for the user. Count what it had to do toward the tier: a key copied from a dashboard is B, reading docs to wire two keys is C.
- For a local run of your own product, firegoat-onboarding's zero-start test measures the same thing. Report its placement as "AX tier list placement" on its own line, never as the checks tier.
