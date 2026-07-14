---
description: Turn a bug ticket or description into a verified, reviewed PR — triage, branch, diagnose, fix, test, pre-PR check, lint, CodeRabbit-style review, and open the PR.
argument-hint: "<clickup-url | ticket-id | \"plain bug description\"> [--autonomous] [--no-e2e] [--base <branch>] [--dry-run]"
---

Invoke the **bugsmith** skill from the bugsmith plugin now.

Operator request: $ARGUMENTS

Steps:

1. Parse `$ARGUMENTS` for the **bug source** — a ClickUp URL or task id, a GitHub
   issue URL/number, or a quoted plain-text description — plus any flags
   (`--autonomous`, `--no-e2e`, `--base <branch>`, `--dry-run`).
2. Load repo config per `${CLAUDE_PLUGIN_ROOT}/reference/config.md` (falls back to
   sensible auto-detected defaults if no config file is present).
3. Run the pipeline in `${CLAUDE_PLUGIN_ROOT}/reference/pipeline.md`.

Mode:
- **Default = interactive**: pause at the two checkpoints (confirm the diagnosis
  before writing code; confirm the diff before pushing/opening the PR).
- **`--autonomous`**: run headless — triage-gate first, make safe default
  decisions instead of asking, and emit the machine-readable result
  (`${CLAUDE_PLUGIN_ROOT}/schemas/bugsmith-result.schema.json`) at the end. This
  is the mode VegaTools / cron / CI would drive.

Never push to a protected base branch directly — always work on a fresh branch
and open a PR. Respect the host repo's `CLAUDE.md` and commit conventions.
