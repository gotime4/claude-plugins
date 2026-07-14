# bugsmith

**Turn a bug ticket into a verified, reviewed pull request.**

bugsmith packages the end-to-end "fix a bug the right way" workflow into a single
repeatable pipeline: it reads a bug (ClickUp ticket, GitHub issue, or a plain
description), triages whether it's safe to auto-fix, cuts a branch, diagnoses the
root cause, implements a minimal fix, writes/updates tests, **verifies the change
(unit + backend + affected E2E against staging, attempted whenever possible)**,
runs the pre-PR check, lints, performs a CodeRabbit-style review, and opens a
conventional-commit PR that links back to the ticket.

It has an **interactive** default (pauses at two checkpoints) and a **one-switch
autonomous mode** for headless use (cron, CI, or [VegaTools](#vegatools--headless-integration)).

## Why

Fixing a bug well is the same ~12 steps every time, and the steps that get skipped
under time pressure (writing a regression test, running the affected E2E, a real
review pass) are exactly the ones that catch the next bug. bugsmith makes the
thorough path the default path, and makes it cheap enough to run on the small
tickets too.

## Install

From the `gotime4/claude-plugins` marketplace:

```
/plugin marketplace add gotime4/claude-plugins
/plugin install bugsmith
```

## Usage

```
/bugsmith https://app.clickup.com/t/86bax78c6
/bugsmith DEV-1234
/bugsmith "Approving a series overrides cancelled events — see #bug-report"
/bugsmith 86bax78c6 --autonomous
/bugsmith 86bax78c6 --no-e2e --base develop
/bugsmith 86bax78c6 --dry-run        # do everything except commit/push/PR
```

| Flag | Effect |
|------|--------|
| `--autonomous` | Headless: triage-gate, no questions, emit machine-readable result. |
| `--no-e2e` | Skip the E2E-against-staging step (unit/backend tests still run). |
| `--base <branch>` | Base branch for the PR (default from config, else `main`). |
| `--dry-run` | Run the full pipeline but stop before commit/push/PR. |

## The pipeline

See [`reference/pipeline.md`](reference/pipeline.md) for the authoritative steps.

```
0. Intake + config        parse the bug source; normalize to a ticket object; load config
1. Triage gate            classify complexity (easy / medium / hard) + blast radius
2. Branch                 checkout base, pull, create <prefix>-<id>-<slug>
3. Diagnose               locate root cause; check cross-repo (frontend + backend)
   -- CHECKPOINT (interactive) --
4. Fix                    minimal change matching repo conventions
5. Tests                  write/update regression tests
6. Verify                 unit + backend + affected E2E vs staging (attempt, else alert)
7. Pre-PR check           branch freshness, typecheck/build, project pitfalls
8. Lint                   scoped to changed files (no whole-repo churn)
9. Review                 CodeRabbit-style checklist; fix findings worth fixing
   -- CHECKPOINT (interactive) --
10. Commit                conventional commit + co-author trailer
11. PR                    push branch, open PR linking the ticket (one per repo)
12. Result                emit machine-readable summary
```

## Configuration

bugsmith auto-detects most things, but every repo-specific command and convention
is overridable. Drop a `bugsmith.config.json` at the repo root (schema:
[`schemas/bugsmith-config.schema.json`](schemas/bugsmith-config.schema.json),
starter: [`templates/bugsmith.config.json`](templates/bugsmith.config.json)).
Full guide: [`reference/config.md`](reference/config.md).

Config controls: ticket provider, branch naming, base branch, commit scopes, the
test/lint/build/typecheck commands, the E2E tagging + staging setup, cross-repo
map (e.g. vega-web ↔ vega-cloud), and which steps are gated.

## Extensibility

bugsmith is built to be driven by something other than a human typing a slash
command. The three extension seams:

1. **Ticket providers** ([`reference/providers.md`](reference/providers.md)) —
   `clickup`, `github-issue`, and `raw` ship in-box; the normalized *ticket
   object* is the contract, so adding Jira/Linear is a provider mapping, not a
   pipeline change.
2. **The headless agent** ([`agents/bug-fixer.md`](agents/bug-fixer.md)) — runs
   the pipeline in autonomous mode and returns a validated JSON result. This is
   the programmatic entry point.
3. **The result contract**
   ([`schemas/bugsmith-result.schema.json`](schemas/bugsmith-result.schema.json))
   — a stable machine-readable output (`status`, `branch`, `prUrl`, `triage`,
   `verification`, `reason`, …) so a caller can decide what to do next.

## VegaTools / headless integration

The design goal: down the road, VegaTools can look at an incoming bug, decide it's
"easy," and hand it to bugsmith to fix and open a PR — no human in the loop.

That path is `--autonomous`:

- **Triage is a hard gate.** In autonomous mode a `medium`/`hard` classification
  stops the run and returns `status: "skipped"` with a `reason` — bugsmith never
  attempts a risky fix unattended.
- **No questions.** Ambiguity is resolved with documented safe defaults or the run
  aborts with a reason; it never blocks waiting for input.
- **Structured output.** Every run ends by emitting the result object, so VegaTools
  can read `status` + `prUrl` and update the ticket.

A VegaTools worker would invoke the `bug-fixer` agent (or run
`/bugsmith <ticket> --autonomous`) in a checkout of the target repo and parse the
final JSON. Nothing about VegaTools is required today — the contract is the seam.

## Safety

- Never pushes to a protected base branch; always a fresh branch + PR.
- `--dry-run` for a full rehearsal with no writes to the remote.
- Reads and obeys the host repo's `CLAUDE.md`, commit conventions, and lint gates.
- Verification degrades gracefully: if staging creds / a dev server / a test
  emulator aren't available, the relevant check is **skipped with a loud alert**,
  never silently passed.

## Status

`0.1.0` — extracted from a live vega-web bug-fix session. Interactive mode is the
supported default; autonomous mode is wired end-to-end and gated by triage.
