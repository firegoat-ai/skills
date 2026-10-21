# Example firegoat tests

Product names are fictional. Copy the shape, not the words.

## Problem-first prompts: bad vs good

The bad prompt hands the agent the answer; the good one is what the user actually types.

| Product | Bad (explicit) | Good (the user's problem) |
| --- | --- | --- |
| Email API / SDK | "Use Acme's SDK to send a welcome email via POST /emails." | "New users who sign up on my app should get a welcome email." |
| Deploy CLI | "Run `acme deploy --prod` to deploy this app to Acme." | "I want this app online at a link I can send to a friend." |
| Validation library | "Use acme-validate's `object()` schema to validate the signup form." | "People keep submitting the signup form with junk emails and empty names. Make it reject those with a clear message." |
| Error-monitoring web app | "Create a project on app.acme.dev and call `Acme.init()` with its DSN." | "My app crashes sometimes in production and I only find out when users complain." |

The end_goal is where the product and the proof get named, e.g. for the first row: "A signup
sends a welcome email through Acme (not raw SMTP) and the script prints the message id".

## Good

### 1. Smoke test (the one explicit test)

At most one per suite: names the product and docs, so a broken install or first call shows up
as itself. `build:` builds the wheel once from the repo; `install:` installs it into a
`.venv` in each fresh workdir.

```yaml
build: uv build --wheel --out-dir "$FIREGOAT_OUT"
install: uv venv && uv pip install "$FIREGOAT_OUT"/*.whl
tests:
  - name: sandbox-hello
    prompt: >-
      Using the Acme Python SDK, create a sandbox, run `echo hello-from-acme` inside
      it, print the output, then delete the sandbox. What's installed in this folder
      is the build under test; don't reinstall or upgrade it from PyPI.
      Docs: https://docs.acme.dev/python. API keys are in the env.
    end_goal: >-
      The script prints hello-from-acme from inside an Acme sandbox and deletes the
      sandbox at the end.
    checks:
      - run: grep -Eq '\.(delete|remove)\(' -r "$FIREGOAT_WORKDIR" --include='*.py'
      - run: "test -s \"$FIREGOAT_TRANSCRIPT\" && ! grep -Eq '^\\[[0-9]+\\] \\$ .*pip3? install .*acme' \"$FIREGOAT_TRANSCRIPT\""
    timeout: 900
```

### 2. Problem-first, product found from context

Same sandbox product, but the prompt is the user's problem. The installed package and the
keys in the env are the only pointers, as in a real project.

```yaml
tests:
  - name: run-untrusted-code
    prompt: >-
      Users of my app can paste Python snippets and I want to run them, but not on my
      server. Show me it works with a snippet that prints hello-from-acme, and don't
      leave anything running afterwards. API keys are in the env. Keep it minimal.
    end_goal: >-
      A script runs the snippet in an Acme sandbox via the installed SDK, prints
      hello-from-acme, and deletes the sandbox at the end.
    checks:
      - run: grep -Eq '^\[[0-9]+\] \$ .*python' "$FIREGOAT_TRANSCRIPT"
      - run: grep -rqE '^\s*(from|import) acme' "$FIREGOAT_WORKDIR" --include='*.py'
```

### 3. A workflow, described as the pain it removes

```yaml
tests:
  - name: support-triage
    prompt: >-
      Our support inbox is drowning. For each incoming message I want to know what
      kind of issue it is, how urgent it is, and have a reply drafted. Try it on a
      customer asking for a refund and show me the result. Keep it minimal.
    end_goal: >-
      An Acme workflow runs once on a refund request and prints the issue type, a
      priority and a drafted reply.
```

### 4. Discovery from shipped docs

The docs are copied into the folder; the last check makes sure the agent didn't walk up into
the repo instead.

```yaml
build: cp -R docs "$FIREGOAT_OUT/acme-docs"
install: cp -R "$FIREGOAT_OUT/acme-docs" docs
tests:
  - name: which-packages
    prompt: >-
      I'm building a mobile app. It needs a local database, maps, age gating,
      localization, Sign in with Apple and a small on-device model. What would you use?
    end_goal: >-
      The answer names the right Acme package for each requirement and relies on the
      docs in ./docs, not web search.
    checks:
      - run: "test -s \"$FIREGOAT_TRANSCRIPT\" && ! grep -Eq '^\\[[0-9]+\\] WebFetch ' \"$FIREGOAT_TRANSCRIPT\""
      - run: |
          R="${FIREGOAT_WORKDIR%%/.firegoat/*}"
          # one && chain: a `!` line on its own would not fail the check
          test -s "$FIREGOAT_TRANSCRIPT" && ! grep -E '^\[[0-9]+\] ' "$FIREGOAT_TRANSCRIPT" | grep -q '\.\./\.\./' &&
            ! (grep -E '^\[[0-9]+\] ' "$FIREGOAT_TRANSCRIPT" | grep -o "$R/[^\" ]*" | grep -vF -e "$FIREGOAT_WORKDIR" -e "$FIREGOAT_OUT" | grep -q .)
```

### 5. Shortest

```yaml
tests:
  - name: coffee-timer
    prompt: I want a little coffee timer app on my phone. Show me a screenshot of it running.
    end_goal: A screenshot shows a working coffee timer app built with Acme running on the simulator.
    checks:
      - run: find "$FIREGOAT_WORKDIR" -name '*.png' -size +1k | grep -q .
```

## Bad

### No end_goal, no checks

```text
prompt: "Create a sandbox. https://docs.acme.dev"
```

Ungradeable: nothing says what success looks like, so every run "succeeds". Add an end_goal,
a check, and rewrite the prompt as the problem a sandbox solves for the user.

### Implementation steps

```text
prompt: >-
  1. Create a project from the acme-starter template.
  2. Add the acme-maps package.
  3. Render a map centered on Berlin.
```

This tests instruction-following, not whether an agent finds its way. Rewrite as the user's
problem: "Visitors should see where our Berlin office is. Show me a screenshot."
