# Verification

One unified verification flow. It detects which layers the change touches and runs
the matching checks. The governing rule:

> **Attempt every applicable check whenever the environment allows it. When it
> can't run, skip it and alert loudly — a skipped check must never read as a
> pass.**

Record each check in the result's `verification` array:
`{ check, layer, status: "passed"|"failed"|"skipped", detail }`.

## 1. Detect the touched layers

From the diff:

- **frontend** — changes under the app source (e.g. `src/**` `.ts`/`.vue`).
- **backend** — changes under a sibling/backend repo declared in config (e.g.
  vega-cloud Go services), or a backend fix identified in Step 3.
- A fix can touch both. Run the checks for **every** layer it touches. There is no
  separate "frontend pipeline" vs "backend pipeline" — it's one flow that fans out
  to whatever the diff hit.

## 2. Unit / component tests (always)

For each touched layer, run **only the affected tests** (not the whole suite):

- frontend: the repo's unit runner (e.g. `npx vitest run <files>`).
- backend: the repo's unit runner (e.g. `go test ./<pkg>/...`, under the emulator
  if the repo requires one).

A **failed** unit test means the fix is wrong — go back and fix it. Re-run after
edits. Only widen scope once the targeted tests pass.

## 3. Backend tests (attempt when backend is touched)

If the fix touches a backend repo, attempt its tests even when the primary change
was frontend and the two are linked (e.g. you also added a server-side guard).

- If the toolchain isn't set up (missing runtime/JDK/emulator, no service account,
  etc.), **skip + alert**: record `skipped` with exactly what's missing, and tell
  the user. Do not fail the run solely because the backend suite couldn't start —
  but be explicit that backend coverage did not run.

## 4. Affected E2E vs staging (attempt unless `--no-e2e`)

This is the step most likely to be un-runnable locally; attempt it, degrade
gracefully.

1. **Select** the affected specs. If the repo has an affected-tags script
   (config `e2e.affectedTagsCmd`, e.g. `e2e/scripts/get-affected-tags.ts`), run it
   against the diff to get the tags/specs. Note which dedicated Playwright
   projects those specs belong to.
2. **Preflight the environment.** Check for: the E2E credentials/secret, a staging
   config, and a browser binary. Start or reuse the local dev server on the
   configured port and mode (staging), and confirm it serves.
3. **Run** the affected specs against the staging backend
   (`TEST_ENV=stage WEBADMIN_URL=http://localhost:<port> ...`), using the correct
   `--project` for isolated specs.
4. **Graceful skip.** If creds are absent, the dev server won't start, or the
   browser isn't installed and can't be installed, record `skipped` with the exact
   blocker and **alert the user**. If the browser is merely missing, it's fine to
   install it once, then run.

Interpreting results: a real **failure** in an affected spec blocks the PR (fix
it). "Couldn't run" is `skipped`, surfaced prominently — in autonomous mode it
goes in the result and the PR body, never hidden.

## 5. Optional: drive the change once

For a fix with real runtime surface, if a `verify`/`run` skill exists in the host
repo, use it to exercise the actual flow once (not just tests). Skip for
docs/test-only diffs.

## Reporting

Summarize verification honestly in both the checkpoint (interactive) and the PR
body: what passed, what was skipped and why. The phrase to avoid is an unqualified
"tests pass" when the E2E or backend suite was actually skipped.
