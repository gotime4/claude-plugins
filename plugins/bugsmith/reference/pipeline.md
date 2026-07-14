# bugsmith pipeline (authoritative)

Run these steps in order. Behavior that differs by mode is marked
**[interactive]** / **[autonomous]**. Track a running result object (schema:
`../schemas/bugsmith-result.schema.json`) and fill it in as you go; you emit it in
Step 12.

---

## Step 0 — Intake + config

1. **Parse the bug source** from the arguments using
   [`providers.md`](providers.md). Produce a normalized **ticket object**:
   `{ id, provider, url, title, body, reporter, env, stepsToReproduce, expected, actual, attachments }`.
   - ClickUp URL/id → fetch via the ClickUp MCP tools (task + comments).
   - GitHub issue → fetch via `gh issue view`.
   - Quoted text → `provider: "raw"`, synthesize a title, leave `id` empty.
2. **Load config** per [`config.md`](config.md). Auto-detect if no file. Resolve:
   base branch, branch-name pattern, commit scope list, test/lint/build/typecheck
   commands, the E2E setup, and the cross-repo map.
3. **Record** the parsed ticket + resolved config into the result object.

If the source can't be resolved to a ticket object, stop:
**[interactive]** ask for a URL/description; **[autonomous]** emit
`status: "failed", reason: "unresolved bug source"`.

---

## Step 1 — Triage gate

Classify the bug with [`triage-rubric.md`](triage-rubric.md) into
`easy | medium | hard` plus a blast-radius note. Record `triage` in the result.

- **[autonomous] Hard gate.** If not `easy`, **stop now**: emit
  `status: "skipped"` with the classification and `reason`. Do not branch, do not
  edit. This is the safety valve for unattended runs.
- **[interactive]** Present the assessment. If `medium`/`hard`, recommend
  proceeding with care or handing back; continue when the human agrees.

---

## Step 2 — Branch

1. `git status` — if the tree is dirty with unrelated changes, do **not** stash or
   revert; **[interactive]** ask, **[autonomous]** abort with a reason. (Untracked
   scratch files unrelated to the fix are fine.)
2. Checkout the base branch, pull fast-forward, and cut a new branch named per
   config (default `<prefix>-<ticketId>-<slug>`, e.g.
   `CU-86bax78c6-series-approval-overrides-cancelled`). For `raw` tickets with no
   id, use a slug of the title.
3. Record `branch` and the base SHA.

---

## Step 3 — Diagnose

1. Map the code involved. For broad searches use the `Explore` agent (or
   `general-purpose`) so you keep the conclusion, not a pile of file dumps. Find:
   the entry points, the state/store/service/repo layers, the exact line(s) where
   the bug originates, and the relevant types/enums.
2. **Cross-repo check.** If the config declares a backend/sibling repo and the bug
   could originate or be enforced there, read the relevant handler/service to
   confirm where the true fix belongs. (A client-only fix that a server undoes is
   not a fix.) Decide the fix's repo(s) and whether a backend change is *required*
   vs a *defense-in-depth follow-up*.
3. Write a crisp **root-cause statement** and the intended minimal fix. Record it.

### ▶ CHECKPOINT 1
**[interactive]** Present root cause + planned fix + which file(s)/repo(s). Get an
OK before writing code. **[autonomous]** skip.

---

## Step 4 — Fix

Implement the **smallest** change that resolves the root cause. Match the
surrounding code's naming, comment density, and idioms. No drive-by refactors. If
the fix is centralizable (one function many callers touch), prefer that over
patching each call site — but also check for *sibling* code paths that bypass the
central function and fix them too.

---

## Step 5 — Tests

Write or update a **regression test** that encodes the exact bug scenario and
would fail on the pre-fix code. Follow the repo's test conventions (framework,
location, mock/factory patterns). Prefer a focused unit/integration test over a
heavy E2E when the logic is unit-testable. Record the test file(s) added.

---

## Step 6 — Verify

Per [`verification.md`](verification.md). One unified flow; it auto-detects which
layers the diff touches and runs the matching checks, **attempting each whenever
possible and skipping-with-alert when the environment can't support it**:

- **Unit/component tests** — always, for every layer the diff touches (e.g.
  `vitest` for frontend, `go test` for backend). Re-run only the affected tests.
- **Backend tests** — if the fix touches a backend/sibling repo, attempt them
  (e.g. Go tests under the Firestore emulator). If the toolchain/emulator isn't
  available, **skip + alert** with what's missing.
- **Affected E2E vs staging** — unless `--no-e2e`: compute the affected specs
  (repo's affected-tags script if present), start/reuse the local dev server, and
  run them against the staging backend. If creds / server / browser aren't
  available, **skip + alert** — never treat "couldn't run" as "passed".

Record per-check outcome in `verification`: `passed | failed | skipped` + why. A
**failed** test (not skipped) means the fix is wrong: return to Step 4.

---

## Step 7 — Pre-PR check

Prefer the repo's own pre-PR check (skill or `scripts/pre-pr-check.sh`); else do
the equivalent: branch freshness vs base, full typecheck/build, and any
project-specific pitfalls the repo documents. Warnings that live in files you did
**not** touch are pre-existing — report them, don't fix them here. Record result.

---

## Step 8 — Lint

Prefer the repo's lint skill/script. **Scope to the files you changed** — do not
run a whole-repo autofix that drags unrelated files into the diff (many repos have
non-canonical formatting in untouched files). Fix lint issues in your files only.

---

## Step 9 — Review

Do a CodeRabbit-style pass with [`review-checklist.md`](review-checklist.md)
(prefer the repo's `code-review` skill if it has one). Read the **full** changed
files, not just the hunks. Group findings by severity. **Fix the Critical/Major
findings and any Minor you judge worth fixing**; for the ones you skip, say why
(intentional scope, unreachable edge). Re-run affected tests after any fix.

### ▶ CHECKPOINT 2
**[interactive]** Show the final diff summary + verification results + review
outcome. Get an OK before pushing. **[autonomous]** skip.

If `--dry-run`: stop here. Emit `status: "dry_run"` with everything you would have
committed.

---

## Step 10 — Commit

Stage only your changed files (never `git add -A`). Write a **conventional commit**
(`type(scope): summary`) using the repo's scopes; reference the ticket
(`Fixes <id>`); add the co-author trailer the repo/CLAUDE.md specifies. Template:
`../templates/commit-message.template.txt`. Let the pre-commit hook run; if it
fails, fix and retry.

---

## Step 11 — PR

`git push -u origin <branch>`, then open the PR against the base branch with `gh`.
Body from `../templates/pr-body.template.md`: summary, repro, root cause, the fix,
testing evidence (unit + verification results, honest about skips), and any
follow-up you deliberately left. Link the ticket. End the body with the
repo/CLAUDE.md-mandated attribution line. One PR per repo if the fix spans repos;
cross-link them. Record `prUrl`.

---

## Step 12 — Result

Emit the machine-readable result object
(`../schemas/bugsmith-result.schema.json`): `status`, `ticket`, `triage`,
`branch`, `prUrl`, `filesChanged`, `verification`, `review`, `reason`,
`followUps`. **[autonomous]** this is the return payload the caller parses.
**[interactive]** also give the human a short prose summary + the PR link.

`status` values: `opened_pr | dry_run | skipped | failed | needs_human`.
