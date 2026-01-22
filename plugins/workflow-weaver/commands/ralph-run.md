# /ralph-run - Execute Tasks

```bash
/ralph-run <task-id>
/ralph-run --all
/ralph-run --phase=N
```

## Options

| Flag | Description | Default |
|------|-------------|---------|
| `--model` | Sub-agent model: `inherit\|haiku\|sonnet\|opus` | `inherit` |
| `--max-loops` | Max retries before escalate | `5` |
| `--fail-fast` | No retries (`--max-loops=1`) | - |
| `--dry-run` | Preview without executing | - |
| `--commit` | Auto-commit on success | - |
| `--verbose` | Detailed logging | - |

## The Loop

```
ATTEMPT → VERIFY → [fail?] → REFLECT → ITERATE
                      ↓
              [max loops?] → ESCALATE
```

Each iteration:
1. **Attempt**: Write code per spec
2. **Verify**: Run `validation_command`
3. **Reflect**: Analyze error, propose fix
4. **Iterate**: Apply fix, retry

## Exit Conditions

| Condition | Action |
|-----------|--------|
| Validation passes | Gatekeeper review → verify |
| Max loops reached | Escalate with history |
| Agent blocked | Escalate with reason |

## Gatekeeper Checks

Before marking verified:
- [ ] Diff within scope
- [ ] No security violations
- [ ] Constraints followed
- [ ] Validation passes

See `skills/weaver/ralph-executor.md` for details.
