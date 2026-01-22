# /weave-status - Check Thread Status

## Invocation
```
/weave-status
```

## Behavior

When invoked, displays:

1. **Thread Overview**
   - Thread ID and original requirement
   - Current status (analyzing/designing/implementing/verifying/blocked)
   - Time elapsed since thread start

2. **Task Progress**
   - List of all tasks with status indicators
   - Currently active task highlighted
   - Completion percentage

3. **Health Indicators**
   - Boundary violation count (with threshold warning)
   - Test failure count
   - Context overflow events

4. **Last Checkpoint**
   - When it was created
   - What state was preserved
   - Resume instructions if interrupted

## Example Output

```
=== Workflow-Weaver Status ===

Thread: abc-123-def
Requirement: "Add user authentication with JWT tokens"
Status: IMPLEMENTING (3/5 tasks complete)
Started: 15 minutes ago

Tasks:
  [x] Analysis - requirement clarification
  [x] Design - token flow architecture
  [x] Implement - auth middleware
  [ ] Implement - token refresh endpoint  <-- ACTIVE
  [ ] Verify - integration tests

Health:
  Violations: 0/3 (OK)
  Test Failures: 0/3 (OK)

Last Checkpoint: "Completed auth middleware"
  Created: 5 minutes ago
  Files preserved: src/middleware/auth.ts
```

## Options

- `/weave-status --verbose`: Include full task details
- `/weave-status --history`: Show completed threads
