# Triage rubric

Classify every bug before touching code. In autonomous mode this is a **hard
gate**: only `easy` proceeds unattended. In interactive mode it sets expectations
and flags where to be careful.

Output: `{ complexity: "easy"|"medium"|"hard", confidence: 0-1, blastRadius, rationale }`.

## easy — safe to auto-fix

All of these hold:

- **Localized.** The fix is plausibly a handful of lines in 1–3 files, in one
  repo (or one repo + a small, clearly-scoped sibling guard).
- **Understood.** The root cause is identifiable from reading the code; the
  reproduction steps are concrete; expected vs actual is unambiguous.
- **Testable.** You can write a regression test that fails before and passes
  after, without standing up elaborate fixtures.
- **Low blast radius.** No schema/migration, no auth/permission model change, no
  public API contract change, no shared utility used in dozens of places, no
  concurrency/transaction redesign.
- **Reversible.** A wrong fix would be caught by tests/review, not silently ship
  data corruption or a security hole.

## medium — proceed with care (interactive) / stop (autonomous)

Any one of:

- Spans several files or crosses a frontend↔backend boundary in a non-trivial way.
- Touches shared/widely-used code, or the fix location is ambiguous.
- Needs a judgment call about intended product behavior (the ticket is
  under-specified in a way that changes the fix).
- Requires new test infrastructure to prove.

## hard — do not auto-fix

Any one of:

- Security, auth, or permissions logic; anything handling secrets or PII.
- Data model / schema / migration changes, or anything that could corrupt or lose
  data.
- Concurrency, race conditions, transactions, or ordering guarantees.
- Public/consumed API or wire-contract changes.
- "Bug" that is actually a feature request, a design decision, or reproduces only
  intermittently with unknown cause.

## Gate behavior

- **autonomous:** `easy` → continue. `medium`/`hard` → stop, `status: "skipped"`,
  put the classification + rationale in `reason`. Never partially attempt.
- **interactive:** always show the classification. For `medium`/`hard`, recommend
  the safer path (careful proceed, or hand back to a human) and continue only when
  the operator agrees.

## Confidence

If `confidence < 0.6` even for an `easy` call, treat it as `medium` for gating
purposes — uncertainty about the classification is itself a risk signal.
