# /ralph-run - Execute Tasks via Ralph Wiggum Loop

## Invocation
```
/ralph-run [task-id] [options]
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
┌─────────────┐
│   REFLECT   │ ← Analyze error, identify fix
└──────┬──────┘
       ▼
┌─────────────┐
│   ITERATE   │ ← Apply fix, go again
└──────┬──────┘
       │
       └──────► Back to ATTEMPT
              (max 5 iterations)
```

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

| Option | Description |
|--------|-------------|
| `--all` | Run all pending tasks sequentially |
| `--phase=N` | Run all tasks in phase N |
| `--max-iterations=N` | Override default max (5) |
| `--dry-run` | Show execution plan without running |
| `--resume` | Continue from last checkpoint |
| `--verbose` | Detailed logging |
| `--commit` | Auto-commit on successful completion |
| `--force` | Re-run already completed task |

## Exit Conditions

| Condition | Status | Action |
|-----------|--------|--------|
| Validation passes | `SUCCESS` | Gatekeeper review |
| Max iterations | `MAX_ITERATIONS` | Escalate to human |
| Agent blocked | `BLOCKED` | Escalate with reason |
| Low confidence 3x | `STUCK` | Escalate to human |
| Gatekeeper rejects | `REJECTED` | Show diff, await fix |

## Output

- Task status updated in `WEAVER_STATE.json`
- Completion report archived to `.claude/skills/weaver/archive/completions/`
- Execution log to stdout
