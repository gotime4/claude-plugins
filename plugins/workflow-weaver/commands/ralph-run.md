# /ralph-run - Execute Tasks via Ralph Wiggum Loop

## Invocation

```bash
# Basic
/ralph-run [task-id]

# With loop limit (stop after N failed attempts)
/ralph-run T3 --max-loops=3

# Run all tasks with limited retries
/ralph-run --all --max-loops=2

# Fail fast (no retries)
/ralph-run T3 --max-loops=1
```

## Overview

The Ralph Executor manages autonomous task execution through isolated sub-agents. Named after Ralph Wiggum's persistent optimism—"I'm learnding!"—each agent iterates through attempt-verify-reflect cycles until success.

## The Ralph Wiggum Loop

```
┌─────────────┐
│   ATTEMPT   │ ← Write code based on spec
└──────┬──────┘
       ▼
┌─────────────┐
│   VERIFY    │ ← Run validation_command
└──────┬──────┘
       │
   Success? ──Yes──► Done
       │
       No
       ▼
   Loops < max? ──No──► Escalate to human
       │
      Yes
       ▼
┌─────────────┐
│   REFLECT   │ ← Analyze error, identify fix
└──────┬──────┘
       ▼
┌─────────────┐
│   ITERATE   │ ← Apply fix, go again
└──────┬──────┘
       │
       └──────► Back to ATTEMPT
```

## Loop Control

Control how many times the agent retries on validation failure:

```bash
# Default: 5 loops
/ralph-run T3

# Conservative: 2 loops then escalate
/ralph-run T3 --max-loops=2

# Aggressive: 10 loops before giving up
/ralph-run T3 --max-loops=10

# Fail fast: no retries, escalate immediately on failure
/ralph-run T3 --max-loops=1

# Set for entire run
/ralph-run --all --max-loops=3
```

**When to use fewer loops:**
- Task is well-defined and should pass quickly
- You want to review failures manually
- Debugging a specific issue

**When to use more loops:**
- Complex task with multiple potential failure points
- Tests are flaky
- You trust the agent to self-correct

## Process Isolation

Each task runs in a **stateless sub-agent** with:

| Boundary | Enforcement |
|----------|-------------|
| File Access | Only `target_files` from task spec |
| Context | Only `required_context` files (~50k tokens max) |
| State | No knowledge of other tasks or system state |
| Duration | 5 min per iteration, 30 min total |

## Gatekeeper Handoff

After validation passes, the Lead Weaver performs:

1. **Diff Review** - Changes within declared scope?
2. **Security Scan** - No forbidden patterns?
3. **Constraint Check** - All rules followed?
4. **Revalidation** - Tests still pass?
5. **Markers Verified** - All success markers green?

Only after ALL checks pass is the task marked `verified` in WEAVER_STATE.json.

## Usage Examples

```bash
# Run a specific task
/ralph-run T3

# Run all pending tasks in order
/ralph-run --all

# Run a specific phase
/ralph-run --phase=2

# Preview without executing
/ralph-run T3 --dry-run

# Continue from last checkpoint
/ralph-run --resume

# Verbose output
/ralph-run T3 --verbose

# Auto-commit on success
/ralph-run T3 --commit
```

## Options

| Option | Description | Default |
|--------|-------------|---------|
| `--max-loops=N` | Max validation retries before escalating | `5` |
| `--all` | Run all pending tasks sequentially | - |
| `--phase=N` | Run all tasks in phase N | - |
| `--dry-run` | Show execution plan without running | - |
| `--resume` | Continue from last checkpoint | - |
| `--verbose` | Detailed logging | - |
| `--commit` | Auto-commit on successful completion | - |
| `--force` | Re-run already completed task | - |
| `--fail-fast` | Alias for `--max-loops=1` | - |

## Exit Conditions

| Condition | Status | Action |
|-----------|--------|--------|
| Validation passes | `SUCCESS` | Gatekeeper review |
| Max loops reached | `MAX_LOOPS` | Escalate to human with history |
| Agent blocked | `BLOCKED` | Escalate with reason |
| Low confidence 3x | `STUCK` | Escalate to human |
| Gatekeeper rejects | `REJECTED` | Show diff, await fix |

## What Happens at Max Loops

When `--max-loops` is reached without success:

```
⚠ Task T3 failed after 3 loops

Loop History:
  Loop 1: TypeError - 'user' is undefined
          Fix attempted: Added null check
  Loop 2: Test assertion failed - expected 200, got 401
          Fix attempted: Added auth header
  Loop 3: Test assertion failed - expected 200, got 401
          Fix attempted: Changed token format

Escalating to human review.

Options:
  1. Review the error and provide guidance
  2. Increase loops: /ralph-run T3 --max-loops=5
  3. Skip task: /ralph-run T4
  4. Abort: /weave-abort
```

## Output

- Task status updated in `WEAVER_STATE.json`
- Completion report archived to `.claude/skills/weaver/archive/completions/`
- Execution log to stdout
