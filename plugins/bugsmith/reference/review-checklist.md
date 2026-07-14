# Review checklist (CodeRabbit-style)

Use this for Step 9 when the host repo has no `code-review` skill of its own (if it
does, prefer that — it encodes the team's learned patterns). Read the **full**
changed files, not just the diff hunks; many issues are only visible in context
(imports, shebangs, how a function fits the file).

Check **every** category for **every** changed file. Group findings as
**Critical** (will cause bugs/security/data issues), **Major** (breaks in edge
cases or violates important patterns), **Minor** (style/naming/unreachable edge).
For each: file:line, what, why it matters, and a suggested fix. Fix Critical/Major
and any Minor worth fixing; note what you skip and why.

## A. Security
- Injection: `execSync`/`exec`/`spawn`/query construction with interpolated
  external input — must be validated/escaped.
- XSS: `v-html`, template literals in HTML context, unsanitized input to the DOM.
- Secrets: hardcoded keys/tokens/passwords; `.env`/credential files being committed.
- Auth: missing permission checks, exposed endpoints, token handling.
- PII in logs: emails/phones/org names logged in code that runs in CI. Mask before
  logging.

## B. Correctness / logic
- Empty match-everything patterns (e.g. an empty grep/filter that matches all).
- Off-by-one: indexing, pagination, loop bounds.
- Race conditions: uncoordinated concurrent async.
- Null/undefined access without guards; CLI `args[++i]` without a bounds check.
- **Flag/option consistency**: a parsed flag must be respected in *all* code paths.
- **Enum/alias exhaustiveness**: if `X` is handled, sibling values/aliases should be
  too; when one function gains a case, check its siblings gained it as well.
- **Sibling code paths**: when you fix logic in one place, search for other paths
  that do the same thing and bypass your fix (this bug's own lesson).

## C. Error handling
- Bare `catch {` — use `catch (error: unknown)` with `instanceof Error` narrowing.
- Silent failures: `.catch(() => null)` / swallowing and returning defaults.
- Missing context: logging a generic message without the actual error.

## D. Types
- `any` — use specific types.
- Assertions (`as X`) without narrowing.
- `interface` vs `type`: follow the repo's convention if one is declared.

## E. Date/time
- Never `toISOString()` for a *local* date — it shifts by a day off-UTC. Use local
  components.
- Explicit about timezone assumptions.

## F. Tests (incl. Playwright/E2E)
- Regression test actually fails pre-fix and passes post-fix.
- Tag syntax `{ tag: ['@x'] }`, not in the title string.
- `.locator('x').filter({ hasText })` over fragile template-literal / `.or()` chains.
- Avoid consecutive `waitForTimeout`; prefer `waitForSelector`/`waitForResponse`.
- Use Page Object Model methods, not raw selectors, when a POM exists.
- Resources created by a test are cleaned up.

## G. Shell / scripts
- Shebang for tsx is `#!/usr/bin/env -S npx tsx` (the `-S`).
- Value-taking flags validate the next arg exists and isn't another flag.
- Git refs from args validated against a safe charset before interpolation.

## H. Style / quality
- `.forEach((x) => { fn(x); })` with braces (avoid implicit-return shorthand).
- Use the project logger, not `console.log` (scripts exempt).
- No dead code / unused vars / commented-out blocks.
- Descriptive, differentiated test ids and names.

## I. CI / workflows
- `needs:` lists all real prerequisites when a job is added.
- Test/build jobs upload artifacts on failure.
- Referenced secrets actually exist.

## J. Docs / comments
- Comments match what the code does.
- Exported/public functions have doc comments.

## Also call out the positives
Note what's done well (clean minimal diff, good guard, thorough test). A review
that only lists faults is less useful and less trusted.
