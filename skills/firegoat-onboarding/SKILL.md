---
name: firegoat-onboarding
description: >-
  Draft 2-3 firegoat evals (firegoat.yaml) for the current repo: tests a coding agent
  runs locally with `npx firegoat eval`, each with a prompt, a goal and script checks.
  Use when the user says "write firegoat evals", "draft evals", "set up firegoat",
  "firegoat init", or asks for local agent-experience (AX) evals of their SDK, CLI,
  API or docs.
---

# Draft firegoat evals

`npx firegoat eval` runs a coding agent (Claude Code) in a fresh, sandboxed workdir with one
prompt, then runs your checks there. The workdir has no repo files. Your job: write 2-3 tests
into `firegoat.yaml` that tell the maintainer whether an agent can succeed with their
product the way a new user would.

**Prompt-writing principles live in the `suggesting-eval-prompts` skill**: read its "Research
the product first" and "What makes a good prompt" sections (end-goal not steps, a realistic
first-run task, one outcome). Ignore its MCP tools (`list_prompts`, `create_prompt`, ...):
here the prompts go into `firegoat.yaml` in this repo. This skill covers only the firegoat
mechanics.

## 1. Learn the product from the repo (read, don't run)

`README*`, the docs dir (`docs/`, `*.mdx`; quickstarts first), package manifests
(`package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`) for install names and entry points,
`examples/`, and an existing `firegoat.yaml` (keep its tests, see step 4). Know: the public docs
URL, the first thing a new user wants working, and the credentials it needs (env var names only).

## 2. Write each prompt for an empty, isolated workdir

On top of the `suggesting-eval-prompts` principles:
- **Observable proof** the checks can verify: a marker printed to stdout, a file, a screenshot.
- **"Keep it minimal."** so the agent doesn't build a demo app.
- **Credentials as env var names**, never values: "Credentials are in env vars ACME_API_KEY and ACME_API_URL."
- **The public docs URL** as the hint (omit it if the test is about discovery). The agent cannot
  read the repo.
- **Local build under test**: with `setup:` / `workdir_setup:` (step 4), say "The package is
  already installed from the build under test. Do not install or upgrade it from the registry."

## 3. Write the goal and checks

`goal`: one sentence a judge could check from the transcript and workdir. `rubric:` (optional
list) adds criteria. You grade both after the run (step 5); firegoat does not.

`checks`: shell commands run in the agent's workdir after the run; exit 0 = pass. Env:
`FIREGOAT_WORKDIR` (the workdir), `FIREGOAT_SESSION` (the agent's stream-json transcript, one
JSON event per line), `FIREGOAT_OUT` (this run's copy of setup's outputs) and any `vars`.
Prove the outcome and catch cheating:
- The marker exists: `grep -rq hello-from-acme "$FIREGOAT_WORKDIR"`.
- The real thing was used: `grep -q '"command":"[^"]*acme ' "$FIREGOAT_SESSION"`.
- The local build was used, not a registry install:
  `! grep -Eq '"command":"[^"]*(pip install|npm (install|i) [^"]*acme@)' "$FIREGOAT_SESSION"`.
- Only local docs were used (when that's the point): `! grep -q '"name":"WebFetch"' "$FIREGOAT_SESSION"`.

Match tool-call JSON (`"command":"...`), not plain words, so the prompt text or the agent saying
"I won't pip install" doesn't trip a check. Quotes inside a command arrive escaped (`\"`), so
`"command":"[^"]*` stops at the first quoted argument: match what comes before it. Keep checks self-contained (one-liners or a `run: |`
block); never reference repo files by relative path. Plain-words criteria go in `rubric:`.

## 4. Write firegoat.yaml

```yaml
agents: [claude-code]      # only claude-code for now
runs: 1                    # runs per test
setup: npm pack --pack-destination "$FIREGOAT_OUT"   # optional: once per eval, in the repo;
                           # $FIREGOAT_OUT is copied into each run and put on the agent's PATH
workdir_setup: npm init -y >/dev/null && npm install "$FIREGOAT_OUT"/*.tgz
                           # optional: in each fresh workdir before the agent starts
vars:                      # optional: defaults for {{vars.KEY}}; `--var KEY=value` overrides
  SHA: main
tests:
  - name: quickstart       # letters, digits, '.', '_', '-'; unique
    prompt: >-
      ...
    goal: ...
    rubric:                # optional; graded by you after the run, not by firegoat
      - ...
    checks:                # optional
      - run: grep -rq some-marker "$FIREGOAT_WORKDIR"
    timeout: 600           # optional, seconds
```

`setup:` and `workdir_setup:` take one shell command or a list. Python:
`setup: uv build --wheel --out-dir "$FIREGOAT_OUT"` and
`workdir_setup: uv venv && uv pip install "$FIREGOAT_OUT"/*.whl`. A CLI binary written to
`$FIREGOAT_OUT` is on the agent's PATH as is. Add them only when the tests should use the repo's
local build.

If `firegoat.yaml` exists, merge: keep every existing test and top-level key, append yours, and
suffix yours with `-2` on a name collision. Never delete or rewrite user tests. Use `>-` folded
blocks for prompts so colons and quotes stay valid YAML.

Before finishing, re-read the file: it parses as YAML, every test has a non-empty `name`,
`prompt` and `goal`, names are unique and match `^[A-Za-z0-9._-]+$`, and no credential values
appear anywhere.

## 5. Run and grade

The human runs `npx firegoat claude login` once (it opens a browser; or
`npx firegoat init --auth api-key` with an Anthropic API key). Ask them; don't wait silently.
Then run `npx firegoat eval --json` yourself. A run takes minutes.

firegoat runs only the script checks; they set the exit code (0 pass, 1 a check failed, 2 not
graded, 3 config error). **You grade the goal and rubric.** For each run, read
`.firegoat/runs/<runId>/<test>/<n>/run.json` (prompt, goal, rubric, check results, paths), then:
- `session` (stream-json, one event per line): the last line is `type: "result"` with the
  agent's final message; tool calls are `tool_use` items in `assistant` messages, their output
  in the following `user` messages. Did it reach the goal with the product, or fake it, give up,
  or ask for help?
- `workdir`: the files it made. Run them if that's how the goal is proven.
- Each rubric item: met or not, with the evidence.

Record one verdict per run:
`npx firegoat grade <runId> <test>#<n> --pass|--fail --reason "<what decided it>"`.
A run can pass its checks and still fail the goal (e.g. it hard-coded the output); say so.

## 6. Hand off

Tell the user in a few lines: which tests you added and why, which env vars to export, the
checks and your grades per run, and where the agent got stuck (quote the transcript).

## Examples

`references/examples.md` has good and bad tests with their YAML. Short version:

Good: "Create a new app - a coffee timer. Run it on the simulator and get the screenshot."
Bad: "Create a sandbox. https://docs.acme.dev" with no goal and no checks: every run passes,
so it measures nothing.
