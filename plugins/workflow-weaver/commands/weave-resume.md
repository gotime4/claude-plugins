# /weave-resume - Resume Interrupted Thread

## Invocation
```
/weave-resume
```

## Behavior

When invoked, the Lead Weaver will:

1. **Load State**
   - Read `WEAVER_STATE.json`
   - Verify thread exists and is not complete
   - Identify last checkpoint

2. **Report Recovery Point**
   ```
   Resuming Thread: {thread.id}
   Original Requirement: "{thread.requirement}"
   Last Checkpoint: {checkpoint.description}
   Created: {checkpoint.timestamp}
   ```

3. **Validate Context**
   - Check that referenced files still exist
   - Verify no external changes conflict with saved state
   - Report any discrepancies

4. **Continue Execution**
   - Resume from checkpoint
   - Re-dispatch any in-progress tasks
   - Continue through remaining tasks

## Recovery Scenarios

### Clean Resume
All files intact, no conflicts. Continues seamlessly.

### File Changed Externally
```
WARNING: File changed since checkpoint
  File: src/auth/middleware.ts
  Checkpoint hash: abc123
  Current hash: def456

Options:
  1. Accept current file (may break assumptions)
  2. Show diff and decide
  3. Abort and start fresh
```

### Missing Files
```
ERROR: Required file missing
  File: src/auth/types.ts

Cannot resume. Options:
  1. Recreate file from checkpoint data (if available)
  2. Abort thread
```

## Options

- `/weave-resume --force`: Skip validation, accept current state
- `/weave-resume --from=<checkpoint-id>`: Resume from specific checkpoint
