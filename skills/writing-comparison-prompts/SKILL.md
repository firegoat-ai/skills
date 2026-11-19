---
name: writing-comparison-prompts
description: >-
  How to write an eval prompt where the agent researches two or more products and picks one for an app, like "Research https://e2b.dev and https://daytona.com and choose which is better for my app": neutral wording, the end goal, checks, one prompt vs competitor lanes, and reading the pick rate across runs. Use when authoring a choice or comparison eval, measuring whether agents pick your product over a competitor, or reading the results of one.
---

# Writing comparison prompts

A comparison prompt asks the agent to look at several products and choose one for a described app. It measures what an agent concludes about your product next to a competitor's, using only what it can find and read. That is a different question from "how well does the agent use your product", which is what a normal eval and its competitor lanes measure (see comparing-competitors).

## One prompt naming all candidates, not competitor lanes
Competitor lanes run the same task against a different provider each, so each lane only ever sees one product. A choice needs every candidate in front of the same agent, so write one prompt that names all of them. Create it in your own org with `create_prompt` (see suggesting-eval-prompts for the tool) and add no competitor lanes to it.

Use lanes afterwards if you also want the second question answered: a normal build task for your product, with the competitor as a lane.

## Write the prompt
- **Describe a real app, then ask for the choice.** The app's needs are what make the choice decidable: "My app runs untrusted Python from users and needs each run isolated, started in under a second and billed per second. Research https://e2b.dev and https://daytona.com and choose which is better for my app." A bare "which is better" gives the agent nothing to weigh, so it falls back on whatever it already believes.
- **Name every candidate the same way.** Same form (all URLs, or all names), no adjectives, no "we use", no "the new". Don't describe a need only one product has a word for; describe the need in the user's words.
- **Don't hint the answer.** No "especially the pricing page", no feature named in one product's vocabulary, no "consider that X recently added Y". If a fact should decide the choice, it has to be findable in the products' own docs, not in the prompt.
- **Ask for the answer in a fixed place.** "Write the product you picked and why to DECISION.md" lets the judge and you read the same thing on every run. It asks for a format, not an answer.
- **Pick research-only or pick-and-build.** Research-only ends at the decision. Pick-and-build ("…then add it to the app so a user's code runs in it") also measures whether the choice held up once the agent touched the product; an agent that switches products halfway is a strong signal.
- **Make "my app" real when you can.** On a scaffold with an app (see suggesting-eval-prompts for what each one holds) the agent can read the code to judge fit. On `blank`, the app exists only in the prompt text, so the description has to carry it.

## End goal and checks
The end goal is that a choice was made, never which product won: the pick is what you are measuring, not a pass condition.

- `endGoal`: "DECISION.md names exactly one of E2B or Daytona as the choice for the app and gives the reasons."
- Checks, each one outcome that happened or didn't (see suggesting-eval-prompts for the rules):
  - `researched-e2b`: "The agent read at least one page on e2b.dev or E2B's docs during the run."
  - `researched-daytona`: the same for Daytona. Without these, an agent that answers from memory passes.
  - `reasons-cite-docs`: "Each reason in DECISION.md points to a page the agent read during the run."
- A research-only task writes no code, so null the default code checks that grade nothing there (`installed`, `build`, `schema`, `client`, `not_placeholder`, `not_hardcoded`, `api_reachable`). The evaluating-mcp-servers skill lists the defaults and how to turn them off. Keep them for pick-and-build.
- If you want the pick rate on the dashboard, add `chose-<you>`: "DECISION.md picks E2B." Know that every run where the agent picked the other product then shows a failed check, so the checks pass rate stops meaning "did the task go well". Leave it out if anyone reads that number as quality.

## Read the results across runs
- **One run is an anecdote.** Set `runsPerEval` (up to 10) or request several runs, and report the pick rate: "picked E2B in 7 of 10 runs". Split it by harness and model (see harnesses-and-models); models can disagree with each other more than runs of one model do.
- **Read the reasons, not only the pick.** A loss that rests on something your docs say wrong, say badly, or don't say at all is the most useful finding the prompt gives you: it is fixable. Quote the agent's reason and link the trace turn (linking-to-the-dashboard).
- **Check what it actually read.** `list_fetched_urls({ promptId })` shows the pages and searches across the prompt's runs, with outcomes. A product whose pricing or limits page was never reached, or returned a 404, lost on missing evidence, not on merit.
- **Watch for order bias.** Agents can favour the product named first. Add a scenario with the same text and the order swapped (`create_prompt` with `parentPromptId` and new `text`) and compare the two pick rates before you claim a preference.
- **Compare like with like.** Only runs of the same prompt text are comparable; editing the app description changes the question. Start a new prompt rather than rewriting one that has history.
- To write it up, see writing-briefs.
