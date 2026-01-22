# /weave-abort - Abort Current Thread

## Invocation
```
/weave-abort
```

## Behavior

When invoked, the Lead Weaver will:

1. **Confirm Intent**
   ```
   Are you sure you want to abort?

   Thread: {thread.id}
   Requirement: "{thread.requirement}"
   Progress: {completed_tasks}/{total_tasks} tasks complete

   This will:
   - Archive the incomplete thread state
   - NOT revert any file changes already made
   - Allow starting a fresh thread

   Type 'yes' to confirm, anything else to cancel.
   ```

2. **On Confirmation**
   - Set `thread.status` to "aborted"
   - Archive to `.claude/skills/weaver/archive/{thread.id}.json`
   - Reset active state to null
   - Report what was accomplished before abort

3. **Cleanup Report**
   ```
   Thread aborted and archived.

   Completed before abort:
   - [x] Analysis phase
   - [x] Design phase
   - [x] Implemented: src/auth/middleware.ts

   Files modified (not reverted):
   - src/auth/middleware.ts (new)
   - src/routes/api.ts (modified)

   To revert changes, use git:
     git checkout -- src/routes/api.ts
     rm src/auth/middleware.ts
   ```

## Options

- `/weave-abort --force`: Skip confirmation
- `/weave-abort --revert`: Attempt to revert file changes (requires clean git state)

## Notes

- Aborted threads are archived, not deleted
- File changes are NOT automatically reverted
- Use git to manage any partial changes
- A new thread can be started immediately after abort
