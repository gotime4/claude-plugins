## Summary

Fixes {TICKET_LINK}: {ONE_LINE_WHAT_WAS_BROKEN}.

**Repro:** {STEPS_OR_TRIGGER}

## Root cause

{WHERE_AND_WHY. Name the file(s)/function(s). If a backend/sibling repo was
involved, say what you verified there and whether a backend change is required vs
a deliberate follow-up.}

## The fix

{WHAT_CHANGED, and the principle behind it. Note if the fix was centralized and
whether sibling code paths were also patched.}

## Testing

- **Regression test:** {file} — fails on the pre-fix code, passes after.
- **Unit/component:** {result}
- **Backend:** {result, or "skipped — <what was missing>"}
- **Affected E2E vs staging:** {result, or "skipped — <what was missing>"}
- **Pre-PR check / typecheck / lint:** {result}

> Be honest here: if the E2E or backend suite was **skipped**, say so plainly —
> do not imply full coverage that didn't run.

## Follow-up (out of scope)

{Deliberately-deferred work, e.g. a defense-in-depth backend guard — or "None".}

{ATTRIBUTION_LINE_PER_CLAUDEMD}
