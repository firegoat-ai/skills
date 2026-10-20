# Example firegoat tests

Product names are fictional. Copy the shape, not the words.

## Good

### 1. Core feature end to end, printed proof

An agent framework with scorers. The prompt names the outcome and what to print; the agent
has to find how in the docs.

```yaml
tests:
  - name: agent-with-scorer
    prompt: >-
      Build an agent with a scorer using https://docs.acme.dev/agents. Ask the agent
      to explain recursion in one sentence, then run a scorer that grades whether the
      answer is concise and accurate. Print the agent answer, the numeric score and
      the scorer reason to stdout. Keep it minimal.
    end_goal: >-
      A minimal project runs and prints the agent's answer, a numeric score and the
      scorer's reason.
    checks:
      - run: grep -Eq '"command":"[^"]*(node|npx|tsx|bun) ' "$FIREGOAT_SESSION"
```

### 2. Multi-step workflow, one realistic input

```yaml
tests:
  - name: support-workflow
    prompt: >-
      Using https://docs.acme.dev/workflows, build a three-step workflow: extract the
      issue type from a support message, have an agent draft a response, then add a
      priority label. Run it once on a customer asking for a refund and print the
      final result. Keep it minimal.
    end_goal: >-
      The workflow runs once on a refund request and prints a result containing the
      issue type, a drafted response and a priority label.
```

### 3. Local build under test, cleanup, no registry install

`setup:` builds the wheel once from the repo; `workdir_setup:` installs it into a `.venv` in
each fresh workdir, so the agent finds the package already installed.

```yaml
setup: uv build --wheel --out-dir "$FIREGOAT_OUT"
workdir_setup: uv venv && uv pip install "$FIREGOAT_OUT"/*.whl
tests:
  - name: sandbox-hello
    prompt: >-
      Using the Acme Python SDK, create a sandbox, run `echo hello-from-acme` inside
      it, print the output, then delete the sandbox. The package is already installed
      from the build under test. Do not install or upgrade it from PyPI.
      Docs: https://docs.acme.dev/python. Credentials are in env vars ACME_API_KEY
      and ACME_API_URL.
    end_goal: >-
      The script prints hello-from-acme from inside a sandbox and deletes the sandbox
      at the end.
    checks:
      - run: grep -Eq '\.(delete|remove)\(' -r "$FIREGOAT_WORKDIR" --include='*.py'
      - run: "! grep -Eq '\"command\":\"[^\"]*pip3? install[^\"]*acme' \"$FIREGOAT_SESSION\""
    timeout: 900
```

### 4. Pure user need, discovery

No docs URL, no package names: the test is whether the agent finds the right packages in
the docs you ship. The agent's folder starts empty, so `setup:` and `workdir_setup:` copy the
docs in; the last check makes sure it didn't walk up into the repo instead.

```yaml
setup: cp -R docs "$FIREGOAT_OUT/acme-docs"
workdir_setup: cp -R "$FIREGOAT_OUT/acme-docs" docs
tests:
  - name: which-packages
    prompt: >-
      I'm building a mobile app with Acme. It needs a local database, maps, age
      gating, localization, Sign in with Apple and a small on-device model. Does Acme
      offer packages for these? Which ones would you use?
    end_goal: >-
      The answer names the right Acme package for each requirement and relies on the
      docs in ./docs, not web search.
    checks:
      - run: "! grep -q '\"name\":\"WebFetch\"' \"$FIREGOAT_SESSION\""
      - run: |
          R="${FIREGOAT_WORKDIR%%/.firegoat/*}"
          # one && chain: a `!` line on its own would not fail the check
          ! grep -q '\.\./\.\./' "$FIREGOAT_SESSION" &&
            ! (grep -o "$R/[^\" ]*" "$FIREGOAT_SESSION" | grep -vF -e "$FIREGOAT_WORKDIR" -e "$FIREGOAT_OUT" | grep -q .)
```

### 5. Shortest

```yaml
tests:
  - name: coffee-timer
    prompt: Create a new app - a coffee timer. Run it on the simulator and get the screenshot.
    end_goal: A screenshot shows a working coffee timer app running on the simulator.
    checks:
      - run: find "$FIREGOAT_WORKDIR" -name '*.png' -size +1k | grep -q .
```

## Bad

### No end_goal, no checks

```text
prompt: "Create a sandbox. https://docs.acme.dev"
```

Ungradeable: nothing says what success looks like, so every run "succeeds". Add an
observable outcome ("run echo hello-from-acme inside it and print the output"), an end_goal
and a check.

### Implementation steps

```text
prompt: >-
  1. Create a project from the acme-starter template.
  2. Add the acme-maps package.
  3. Render a map centered on Berlin.
```

This tests instruction-following, not whether an agent finds its way with your docs.
Rewrite as the user's need: "Show a map centered on Berlin in a new Acme app. Keep it
minimal. Take a screenshot."
