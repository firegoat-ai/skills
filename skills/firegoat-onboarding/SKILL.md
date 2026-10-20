---
name: firegoat-onboarding
description: >-
  Draft 2-3 firegoat evals (firegoat.yaml) for the current repo: tests a coding agent
  runs locally with `npx firegoat eval`, each with a prompt, an end_goal and script checks.
  Use when the user says "write firegoat evals", "draft evals", "set up firegoat",
  "firegoat init", or asks for local agent-experience (AX) evals of their SDK, CLI,
  API or docs.
---

# Draft firegoat evals

`npx firegoat eval` runs the user's own Claude Code, with none of their config, in a fresh empty
folder (`.firegoat/runs/<runId>/<test>/<n>/work`) with one prompt, then runs your checks there. Your job: write 2-3 tests
into `firegoat.yaml` that tell the maintainer whether an agent can succeed with their
product the way a new user would.

## 1. Learn the product from the repo (read, don't run)

`README*`, the docs dir (`docs/`, `*.mdx`; quickstarts first), landing copy, package manifests
(`package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`) for install names and entry points,
`examples/`, and an existing `firegoat.yaml` (keep its tests, see step 4). Know: the public docs
URL, the credentials it needs (env var names only), and whether the tool keeps global on-disk
state (a home dir like `~/.acme`, named sessions, fixed ports). Runs go in parallel, so that
state needs a per-run home (step 4).

**Before writing any prompt, write down** (in your notes to the user, not in the yaml):
- **Who the users are** and **what problem the product solves for them**, in their words, from
  the README, landing copy and docs. Not the feature list: "my signup emails land in spam", not
  "transactional email API".
- **The situations** in which such a user reaches for it: what they were doing, what broke or
  was missing. Each test prompt starts from one of these. Only pick problems the docs show the
  product actually solves, so the end goal is reachable.

## 2. Write each prompt as the user's problem

A test asks: can an agent that only knows the user's problem find the product and use it well?
So write the prompt the way a real person types to their coding agent:

- **The problem or goal in their own words.** Vague and underspecified is fine and expected.
  "New users who sign up should get a welcome email", not "use Acme's SDK to send a welcome
  email via POST /emails".
- **No product name, no API, command or package names, no docs URL, no steps.** Those are the
  answer; the test is whether the agent gets there. Exception: if a real user would only ever
  arrive already knowing the product (a migration between its versions, a plugin for it), at
  most one test may name it.
- **At most one smoke test** may be explicit (product, docs URL, first call), so a broken
  install shows up as itself. Every other test is problem-first.
- **The context does the pointing**, as in a real project: the package already installed by
  `workdir_setup:`, docs copied into the folder, keys in the env. Say "API keys are in the env",
  without naming the vars (that names the product).
- **One outcome per test**, so a failure points at one thing.
- **Proof, asked for the way a user would**: "show me it works", "send one to me at
  test@example.com", "save a screenshot", so the checks have something to verify.
- **"Keep it minimal."** so the agent doesn't build a demo app.
- **Local build under test**: with `setup:` / `workdir_setup:` (step 4), add "What's installed in
  this folder is the build under test; don't reinstall or upgrade it from the registry."
- **No isolation**: the folder starts empty but sits in `.firegoat/runs/` inside the repo, so
  the agent could walk up and read source or docs. When it should find things on its own, add
  a check that it stayed in its folder (see step 3).

## 3. Write the end_goal and checks

`end_goal`: what is true when the user's problem is solved, checkable from the transcript and
workdir. The prompt is vague; the end_goal is concrete and may name the product: "A signup
sends a welcome email through Acme's SDK, not raw HTTP, and the script prints the message id". firegoat never
grades it, only hands it over; you judge it after the run (step 5).

`checks`: shell commands run in the agent's workdir after the run; exit 0 = pass. Env:
`FIREGOAT_WORKDIR` (the workdir), `FIREGOAT_TRANSCRIPT` (what the agent did, plain text: each
tool call as one line `[n] $ <command>` or `[n] Write <path>`, then its short output, the agent's
text, its final answer, the files it made), `FIREGOAT_SESSION` (claude's full session file, JSON,
~50x larger, escaped; don't grep it), `FIREGOAT_OUT` (setup's outputs, shared by all runs of one eval),
any `vars` and the run's `env:`.
Prove the outcome and catch cheating:
- The marker exists: `grep -rq hello-from-acme "$FIREGOAT_WORKDIR"`.
- The real thing was used: `grep -Eq '^\[[0-9]+\] \$ .*acme ' "$FIREGOAT_TRANSCRIPT"`.
- The local build was used, not a registry install (see step 4 for the full guard):
  `test -s "$FIREGOAT_TRANSCRIPT" && ! grep -Eq '^\[[0-9]+\] \$ .*(pip install|npm (install|i) .*acme@)' "$FIREGOAT_TRANSCRIPT"`.
- Only local docs were used (when that's the point): `test -s "$FIREGOAT_TRANSCRIPT" && ! grep -Eq '^\[[0-9]+\] WebFetch ' "$FIREGOAT_TRANSCRIPT"`.
- It stayed in its folder (a heuristic: no paths into the repo except its own workdir and
  `$FIREGOAT_OUT`; that also keeps it out of `.firegoat/fixtures/`):
  ```yaml
  - run: |
      R="${FIREGOAT_WORKDIR%%/.firegoat/*}"
      # one && chain: a `!` line on its own would not fail the check
      test -s "$FIREGOAT_TRANSCRIPT" && ! grep -E '^\[[0-9]+\] ' "$FIREGOAT_TRANSCRIPT" | grep -q '\.\./\.\./' &&
        ! (grep -E '^\[[0-9]+\] ' "$FIREGOAT_TRANSCRIPT" | grep -o "$R/[^\" ]*" | grep -vF -e "$FIREGOAT_WORKDIR" -e "$FIREGOAT_OUT" | grep -q .)
  ```

Anchor greps to the transcript's tool-call lines, `^\[[0-9]+\] \$ ` (commands) or
`^\[[0-9]+\] Write ` etc., not plain words: tool output (a README showing `$ npm install acme`)
and the agent's prose ("I won't pip install") must not trip a check. Commands appear
unescaped, exactly as run. Start every negated grep with `test -s "$FIREGOAT_TRANSCRIPT" &&`:
on a missing file grep exits 2 and `!` turns that into a pass. The lines have no `: `, so a grep
fits in an unquoted YAML value.

## 4. Write firegoat.yaml

```yaml
runs: 1                    # runs per test
setup: npm pack --pack-destination "$FIREGOAT_OUT"   # optional: once per eval, in the repo;
                           # $FIREGOAT_OUT is first on the agent's PATH
workdir_setup: npm init -y >/dev/null && npm install "$FIREGOAT_OUT"/*.tgz
                           # optional: in each fresh workdir before the agent starts
vars:                      # optional: defaults for {{vars.KEY}}; `--var KEY=value` overrides
  SHA: main                # setup, workdir_setup and checks only, never the agent
env:                       # optional: the agent's env (also its workdir_setup and checks)
  NONINTERACTIVE_HOME: "/tmp/ni-{{run.id}}-{{test.name}}-{{run.n}}"
tests:
  - name: quickstart       # letters, digits, '.', '_', '-'; unique
    prompt: >-
      ...
    end_goal: ...          # judged by you after the run, not by firegoat
    checks:                # optional
      - run: grep -rq some-marker "$FIREGOAT_WORKDIR"
    timeout: 600           # optional, seconds
    env:                   # optional: on top of the top-level env:
      ACME_BETA_LOGIN: "1"
```

`env:` is the only way to set env vars for the agent; it never sees `vars:`. Use it for per-run
tool home dirs, feature flags, and pointing a tool at the local build. eval runs up to 4 runs at
once (`--parallel N`), so a tool with global state (step 1) gets its home from `env:`, e.g.
`ACME_HOME: "/tmp/acme-{{run.id}}-{{test.name}}-{{run.n}}"` (`{{run.n}}` alone repeats across
tests); if it has no such knob (a fixed port), run with `--parallel 1`. Values are strings (quote
numbers) with placeholders `{{vars.KEY}}`,
`{{run.id}}`, `{{run.n}}`, `{{test.name}}`, plus `$FIREGOAT_OUT` and `$FIREGOAT_WORKDIR` (the
same dirs setup and checks see; no other `$` expansion, `$HOME` stays literal). `PATH`, `CLAUDE_CONFIG_DIR` and
`FIREGOAT_*` are firegoat's and can't be set. Credentials stay in the shell that runs eval, not
in firegoat.yaml.

**Fixtures** (helper scripts, sample data, a fake server the test needs) go in files under
`.firegoat/fixtures/`, never inline in firegoat.yaml as heredocs. That folder is committed with
firegoat.yaml. setup runs in the repo, so copy them from there:
`setup: cp -R .firegoat/fixtures/. "$FIREGOAT_OUT"`, then in `workdir_setup:` copy what a test
needs into its folder (`cp "$FIREGOAT_OUT/wizard.mjs" .`). Files in `$FIREGOAT_OUT` are also on
the agent's PATH.

`setup:` and `workdir_setup:` take one shell command or a list. Python:
`setup: uv build --wheel --out-dir "$FIREGOAT_OUT"` and
`workdir_setup: uv venv && uv pip install "$FIREGOAT_OUT"/*.whl`. A binary written to
`$FIREGOAT_OUT` is on the agent's PATH, but that only helps for CLIs called by bare name.
- **npx**: pack and install the CLI into the workdir (the npm example above); `npx acme` prefers
  `node_modules/.bin`. Add a positive check `test -x "$FIREGOAT_WORKDIR/node_modules/.bin/acme"`.
- **uvx**: it uses its own tool environment and ignores the workdir `.venv` and a project
  `uv.toml`, so point it at the wheel with `env: {UV_FIND_LINKS: "$FIREGOAT_OUT"}`.
  uv takes the highest version from the index and the links, so the
  local build must be newer than the release.
- **pipx run**: test the bare-name binary or `python -m acme` from the workdir `.venv` instead,
  and say so in the prompt.

Guard against the registry release in either case:
`test -s "$FIREGOAT_TRANSCRIPT" && ! grep -Eq '^\[[0-9]+\] \$ .*((npm (install|i|add)|pnpm (add|dlx)) .*acme|(npx|bunx|yarn dlx) .*acme@|uvx .*acme(@|==)|pipx (run|install) .*acme|pip3? install .*acme)' "$FIREGOAT_TRANSCRIPT"`
Don't flag plain `npx acme` or `uvx acme`: with the local build that's the right way to run it
(but `npx acme@latest` skips it, hence `acme@` in the guard), and the positive
`node_modules/.bin` check above proves the local build is there.
Add them only when the tests should use the repo's local build.

If `firegoat.yaml` exists, merge: keep every existing test and top-level key, append yours, and
suffix yours with `-2` on a name collision. Never delete or rewrite user tests. Use `>-` folded
blocks for prompts so colons and quotes stay valid YAML.

Before finishing, re-read the file: it parses as YAML, every test has a non-empty `name`,
`prompt` and `end_goal`, names are unique and match `^[A-Za-z0-9._-]+$`, and no credential values
appear anywhere.

## 5. Run and grade

firegoat runs the user's `claude` on its existing login. If eval says claude is not logged in,
ask the human to run `claude` once and log in; don't wait silently. Run
`npx firegoat eval --json` yourself. A run takes minutes.

firegoat runs only the script checks; they set the exit code (0 pass, 1 a check failed, 2 not
graded, 3 config error). **You judge the end_goal.** For each run, read
`.firegoat/runs/<runId>/<test>/<n>/run.json` (prompt, end_goal, check results, paths), then:
- `transcript` (a few KB): every command the agent ran with its short output, what it said, its
  final answer and the files it made. Did it reach the end goal with the product, or fake it,
  give up, or ask for help? Open `session` (claude's full JSON) only if a step's output was cut
  and you need it.
- `workdir`: the files it made. Run them if that's how the end goal is proven. Each criterion in it: met or not, with the evidence.

firegoat records no verdict beyond the checks: keep your grades yourself (per run: pass/fail
and the evidence). A run can pass its checks and still fail the end goal (e.g. it hard-coded the
output, or solved the problem with a competitor); say so. If every run passes on the first try,
the prompts are probably too explicit.

## 6. Hand off and commit

Tell the user in a few lines: which tests you added and why, which env vars to export, the
checks and your grades per run, and where the agent got stuck (quote the transcript).

Onboarding is finished when the evals are committed, so the team and CI run the same tests.
End by asking the user to commit, and offer to do it: "Commit firegoat.yaml and the firegoat
skill to finish onboarding?" On yes, commit with a message like `chore: add firegoat evals`:
- `firegoat.yaml` and `.firegoat/fixtures/` if you made fixtures (`.firegoat/runs/` and
  `.firegoat/.claude/` are ignored);
- the installed skill, `.claude/skills/firegoat-onboarding/`. If it is a symlink (`npx skills`
  can link it to `.agents/skills/<name>/`), also commit `.agents/skills/` so teammates and CI don't get
  dangling links;
- `skills-lock.json`, if `npx skills` wrote one.

If `.gitignore` ignores `.claude/` (or `.agents/`), say so and ask before `git add -f`.
Never commit `.firegoat/runs/`, `.firegoat/.claude/` (runs, sessions) or credentials.

## Examples

`references/examples.md` has good and bad tests with their YAML. Short version:

Good: "My app crashes sometimes in production and I only hear about it when users complain."
Bad: "Install @acme/node, call Acme.init() with your DSN, then throw a test error": it names the
product and the steps, so it tests instruction-following, not whether an agent finds its way.
