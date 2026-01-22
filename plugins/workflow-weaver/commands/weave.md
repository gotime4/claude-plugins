# /weave - Start Workflow Thread

```bash
/weave <requirement>
/weave --from-plan <path>
/weave --from-issue <github-url>
```

## Options

| Flag | Description |
|------|-------------|
| `--from-plan` | Use plan file as input |
| `--from-issue` | Pull from GitHub issue |
| `--model` | Sub-agent model: `inherit\|haiku\|sonnet\|opus` |
| `--max-loops` | Max retries per task (default: 5) |
| `--fail-fast` | Stop on first failure |
| `--dry-run` | Preview without executing |

## Behavior

1. Parse input (requirement, plan file, or issue)
2. Initialize WEAVER_STATE.json with new thread
3. Analyze complexity and decompose into tasks
4. Execute tasks via ralph-run
5. Checkpoint after each task

## State

Thread state persisted in `WEAVER_STATE.json`. Resume interrupted threads with `/weave-resume`.

See `skills/weaver/SKILL.md` for orchestration details.
