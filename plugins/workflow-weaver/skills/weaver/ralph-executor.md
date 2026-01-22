# Ralph Executor Skill

## Command
```
/ralph-run [task-id]
```

## Overview

The Ralph Executor is the **Loop Manager** of the Workflow-Weaver system. Named after the lovably persistent Ralph Wiggum—who keeps trying no matter what—this skill manages the autonomous execution of tasks through isolated sub-agents that iterate until success.

**Philosophy**: "I'm learnding!" — Each iteration learns from the previous failure.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      LEAD WEAVER                            │
│                   (Orchestrator Layer)                      │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  /ralph-run                                          │   │
│  │  ├── Load tasks.json                                 │   │
│  │  ├── Select next task                                │   │
│  │  ├── Spawn isolated sub-agent ──────────┐           │   │
│  │  ├── Monitor execution                   │           │   │
│  │  ├── Receive completion report           │           │   │
│  │  ├── Gatekeeper review                   │           │   │
│  │  └── Update WEAVER_STATE.json            │           │   │
│  └──────────────────────────────────────────│───────────┘   │
└─────────────────────────────────────────────│───────────────┘
                                              │
                    ┌─────────────────────────▼───────────────────────┐
                    │           ISOLATED SUB-AGENT                    │
                    │              (Stateless)                        │
                    │                                                 │
                    │  Context Window (scoped):                       │
                    │  ├── Task spec from tasks.json                  │
                    │  ├── required_context files ONLY                │
                    │  └── NO access to WEAVER_STATE or other tasks   │
                    │                                                 │
                    │  ┌─────────────────────────────────────────┐   │
                    │  │         RALPH WIGGUM LOOP               │   │
                    │  │                                          │   │
                    │  │  ┌──────────┐                           │   │
                    │  │  │ ATTEMPT  │ Write code/test           │   │
                    │  │  └────┬─────┘                           │   │
                    │  │       ▼                                  │   │
                    │  │  ┌──────────┐                           │   │
                    │  │  │ VERIFY   │ Run validation_command    │   │
                    │  │  └────┬─────┘                           │   │
                    │  │       │                                  │   │
                    │  │       ▼                                  │   │
                    │  │   Success? ──Yes──► Exit loop           │   │
                    │  │       │                                  │   │
                    │  │       No                                 │   │
                    │  │       ▼                                  │   │
                    │  │  ┌──────────┐                           │   │
                    │  │  │ REFLECT  │ Analyze error log         │   │
                    │  │  └────┬─────┘                           │   │
                    │  │       ▼                                  │   │
                    │  │  ┌──────────┐                           │   │
                    │  │  │ ITERATE  │ Adjust implementation     │   │
                    │  │  └────┬─────┘                           │   │
                    │  │       │                                  │   │
                    │  │       └──────► Back to ATTEMPT          │   │
                    │  │              (until max_iterations)      │   │
                    │  └─────────────────────────────────────────┘   │
                    │                                                 │
                    │  Output: Completion Report                      │
                    └─────────────────────────────────────────────────┘
```

---

## Phase 1: Process Isolation

### Sub-Agent Spawning Protocol

When `/ralph-run` initiates a task:

#### 1.1 Context Assembly

```python
def assemble_agent_context(task):
    context = {
        "task_spec": extract_task_only(task),  # No other tasks visible
        "files": {}
    }

    # Load ONLY files in required_context
    for file_path in task.required_context.direct_targets:
        context["files"][file_path] = read_file(file_path)

    for file_path in task.required_context.dependencies:
        context["files"][file_path] = read_file(file_path)

    for exemplar in task.required_context.exemplars:
        context["files"][exemplar.path] = read_file(exemplar.path)

    for util in task.required_context.test_utilities:
        context["files"][util] = read_file(util)

    # Verify context size
    token_count = estimate_tokens(context)
    assert token_count <= 50000, "Context too large"

    return context
```

#### 1.2 Isolation Guarantees

The sub-agent is spawned with these hard boundaries:

| Boundary | Enforcement |
|----------|-------------|
| **File Access** | Can ONLY read/write files in `target_files` |
| **Context** | Cannot see other tasks, WEAVER_STATE, or blueprints |
| **State** | Stateless—no memory of previous tasks |
| **Network** | No external API calls (unless explicitly required) |
| **Duration** | Max runtime per iteration: 5 minutes |
| **Iterations** | Max iterations: 5 (configurable) |

#### 1.3 Sub-Agent Prompt Template

```markdown
# Sub-Agent Task Assignment

You are a stateless implementation agent. Your ONLY job is to complete the task below.
You have NO knowledge of other tasks or the broader system.

## Task Specification

**ID**: {task.id}
**Name**: {task.name}
**Type**: {task.type}

**Description**:
{task.description}

## Target Files

{for file in task.target_files}
- `{file.path}` ({file.action}): {file.purpose}
{endfor}

## Constraints

{for constraint in task.constraints}
- **{constraint.rule}**
  - Reference: `{constraint.source}`
  - Example: `{constraint.example}`
{endfor}

## Validation

Your implementation is correct when this command succeeds:
```bash
{task.validation.command}
```

Expected outcome: {task.validation.expected_outcome}

## Success Markers

{for marker in task.success_markers}
- {marker.type}: {marker.description}
{endfor}

## Context Files

The following files are provided for reference:

{for path, content in context.files}
### {path}
```
{content}
```
{endfor}

## Instructions

1. Implement the changes required to satisfy the task
2. Run the validation command
3. If validation fails, analyze the error and adjust
4. Repeat until validation passes or you've exhausted options
5. Report your results using the completion template

## BOUNDARIES (CRITICAL)

- You may ONLY modify files listed in "Target Files"
- You may NOT create files not listed in "Target Files"
- You may NOT install new dependencies
- You may NOT access files outside your context
- You may NOT modify test expectations to make tests pass
- If you cannot complete the task within constraints, report BLOCKED
```

---

## Phase 2: The Ralph Wiggum Loop

### Loop Implementation

```
FUNCTION ralph_loop(task, context, max_iterations=5):
    iteration = 0
    history = []

    WHILE iteration < max_iterations:
        iteration += 1
        LOG("Ralph Loop: Iteration {iteration}/{max_iterations}")

        # ═══════════════════════════════════════════════
        # STEP 1: ATTEMPT
        # ═══════════════════════════════════════════════
        attempt_result = sub_agent.attempt(task, context, history)

        IF attempt_result.status == "BLOCKED":
            RETURN RalphResult(
                status="BLOCKED",
                iteration=iteration,
                reason=attempt_result.reason,
                history=history
            )

        history.append({
            "iteration": iteration,
            "phase": "ATTEMPT",
            "changes": attempt_result.changes,
            "reasoning": attempt_result.reasoning
        })

        # ═══════════════════════════════════════════════
        # STEP 2: VERIFY
        # ═══════════════════════════════════════════════
        verify_result = execute_validation(
            command=task.validation.command,
            timeout=task.validation.timeout_seconds,
            expected=task.validation.expected_outcome
        )

        history.append({
            "iteration": iteration,
            "phase": "VERIFY",
            "command": task.validation.command,
            "exit_code": verify_result.exit_code,
            "stdout": verify_result.stdout,
            "stderr": verify_result.stderr,
            "success": verify_result.success
        })

        IF verify_result.success:
            LOG("✓ Validation passed on iteration {iteration}")
            RETURN RalphResult(
                status="SUCCESS",
                iteration=iteration,
                history=history
            )

        # ═══════════════════════════════════════════════
        # STEP 3: REFLECT
        # ═══════════════════════════════════════════════
        reflection = sub_agent.reflect(
            task=task,
            attempt=attempt_result,
            verification=verify_result,
            previous_iterations=history
        )

        history.append({
            "iteration": iteration,
            "phase": "REFLECT",
            "error_analysis": reflection.error_analysis,
            "root_cause": reflection.root_cause,
            "proposed_fix": reflection.proposed_fix,
            "confidence": reflection.confidence
        })

        # Check if reflection indicates we're stuck
        IF reflection.confidence < 0.3:
            IF iteration >= 3:
                LOG("⚠ Low confidence after 3 iterations, escalating")
                RETURN RalphResult(
                    status="STUCK",
                    iteration=iteration,
                    reason="Low confidence in fix after multiple attempts",
                    history=history
                )

        # ═══════════════════════════════════════════════
        # STEP 4: ITERATE
        # ═══════════════════════════════════════════════
        # Apply the proposed fix and continue to next iteration
        context = update_context_with_fix(context, reflection.proposed_fix)

        LOG("↻ Iterating with proposed fix: {reflection.proposed_fix.summary}")

    # Max iterations reached
    RETURN RalphResult(
        status="MAX_ITERATIONS",
        iteration=iteration,
        reason="Failed to achieve validation after {max_iterations} attempts",
        history=history
    )
```

### Reflection Protocol

The REFLECT phase is critical for learning from failures:

```markdown
## Reflection Analysis Template

### Error Classification

**Error Type**: [compile|runtime|test_assertion|lint|type|timeout|unknown]

**Error Location**:
- File: {file}
- Line: {line}
- Function/Component: {context}

### Error Analysis

**Raw Error**:
```
{stderr or test output}
```

**Root Cause**:
{Why did this fail? Be specific.}

**Contributing Factors**:
1. {factor 1}
2. {factor 2}

### Proposed Fix

**Strategy**: {What approach to take}

**Specific Changes**:
```diff
- {old code}
+ {new code}
```

**Why This Will Work**:
{Explain the reasoning}

### Confidence Assessment

**Confidence**: {0.0 - 1.0}

**Factors**:
- Have I seen this error pattern before? {yes/no}
- Is the fix isolated? {yes/no}
- Could this break something else? {yes/no}
- Am I repeating a previous failed fix? {yes/no}

### Anti-Loop Detection

**Previous Fixes Attempted**:
{list of fixes from history}

**Is Current Fix Different?**: {yes/no}
**If Similar, Why Try Again?**: {reason or N/A}
```

### Loop Termination Conditions

| Condition | Result | Next Action |
|-----------|--------|-------------|
| Validation passes | `SUCCESS` | Proceed to Gatekeeper |
| Max iterations reached | `MAX_ITERATIONS` | Escalate to human |
| Agent reports blocked | `BLOCKED` | Escalate to human |
| Confidence < 0.3 for 3 iterations | `STUCK` | Escalate to human |
| Same fix attempted twice | `LOOP_DETECTED` | Force new approach or escalate |
| Timeout exceeded | `TIMEOUT` | Escalate to human |

---

## Phase 3: Gatekeeper Handoff

Once a task achieves `SUCCESS` status, the Gatekeeper protocol begins.

### 3.1 Sub-Agent Completion Report

The sub-agent must generate this report before handoff:

```json
{
  "task_id": "T3",
  "status": "SUCCESS",
  "iterations_required": 2,

  "summary": {
    "what_was_done": "Implemented AuthContext provider with login/logout methods",
    "approach_taken": "Used existing ThemeContext as pattern, integrated with API client",
    "challenges_encountered": [
      "Initial type mismatch with User interface",
      "Had to adjust token refresh logic"
    ],
    "lessons_learned": [
      "Token refresh should be handled at API client level"
    ]
  },

  "changes": {
    "files_created": [
      {
        "path": "src/context/AuthContext.tsx",
        "lines": 87,
        "purpose": "Main auth context provider"
      }
    ],
    "files_modified": [
      {
        "path": "src/context/index.ts",
        "lines_added": 1,
        "lines_removed": 0,
        "summary": "Added AuthContext export"
      }
    ]
  },

  "validation_evidence": {
    "command": "npm run test -- src/context/AuthContext.test.tsx",
    "exit_code": 0,
    "output_summary": "Test Suites: 1 passed\nTests: 5 passed",
    "duration_seconds": 3.2
  },

  "success_markers_verified": [
    {
      "marker": "file_exists:src/context/AuthContext.tsx",
      "verified": true
    },
    {
      "marker": "exports_exist:AuthProvider,useAuth",
      "verified": true
    },
    {
      "marker": "command_succeeds:npm run test",
      "verified": true
    }
  ],

  "iteration_history": [
    {
      "iteration": 1,
      "outcome": "FAILED",
      "error": "Type 'string' is not assignable to type 'AuthUser'",
      "fix_applied": "Added proper type casting for user object"
    },
    {
      "iteration": 2,
      "outcome": "SUCCESS",
      "notes": "All tests passing"
    }
  ],

  "confidence": 0.95,
  "notes_for_reviewer": "Consider adding error boundary for auth failures in future task"
}
```

### 3.2 Lead Weaver Gatekeeper Review

The Lead Weaver performs these checks before accepting the work:

```
FUNCTION gatekeeper_review(task, completion_report):
    checks = []

    # ═══════════════════════════════════════════════
    # CHECK 1: Diff Review
    # ═══════════════════════════════════════════════
    diff = git_diff(task.target_files)

    checks.append({
        "name": "DIFF_SCOPE",
        "description": "Changes are within declared scope",
        "pass": all_changes_in_scope(diff, task.target_files),
        "details": {
            "expected_files": task.target_files,
            "actual_files": diff.files_changed
        }
    })

    # ═══════════════════════════════════════════════
    # CHECK 2: No Forbidden Patterns
    # ═══════════════════════════════════════════════
    security_scan = scan_for_forbidden_patterns(diff)

    checks.append({
        "name": "SECURITY_PATTERNS",
        "description": "No hardcoded secrets or dangerous patterns",
        "pass": len(security_scan.violations) == 0,
        "details": security_scan.violations
    })

    # ═══════════════════════════════════════════════
    # CHECK 3: Constraint Adherence
    # ═══════════════════════════════════════════════
    constraint_check = verify_constraints(diff, task.constraints)

    checks.append({
        "name": "CONSTRAINTS",
        "description": "All constraints were followed",
        "pass": constraint_check.all_satisfied,
        "details": constraint_check.results
    })

    # ═══════════════════════════════════════════════
    # CHECK 4: Validation Reproducible
    # ═══════════════════════════════════════════════
    revalidation = execute_validation(task.validation.command)

    checks.append({
        "name": "REVALIDATION",
        "description": "Validation still passes",
        "pass": revalidation.success,
        "details": {
            "exit_code": revalidation.exit_code,
            "output": revalidation.stdout
        }
    })

    # ═══════════════════════════════════════════════
    # CHECK 5: Success Markers
    # ═══════════════════════════════════════════════
    markers_check = verify_all_markers(task.success_markers)

    checks.append({
        "name": "SUCCESS_MARKERS",
        "description": "All success markers verified",
        "pass": markers_check.all_passed,
        "details": markers_check.results
    })

    # ═══════════════════════════════════════════════
    # VERDICT
    # ═══════════════════════════════════════════════
    all_passed = all(c["pass"] for c in checks)

    IF all_passed:
        RETURN GatekeeperResult(
            verdict="APPROVED",
            checks=checks,
            ready_to_merge=True
        )
    ELSE:
        failed_checks = [c for c in checks if not c["pass"]]
        RETURN GatekeeperResult(
            verdict="REJECTED",
            checks=checks,
            ready_to_merge=False,
            rejection_reasons=failed_checks
        )
```

### 3.3 State Update and Merge

On approval:

```python
def complete_task(task, completion_report, gatekeeper_result):
    # Update WEAVER_STATE.json
    weaver_state = load_weaver_state()

    task_entry = find_task(weaver_state, task.id)
    task_entry["status"] = "verified"
    task_entry["completed_at"] = now()
    task_entry["output_artifacts"] = completion_report.changes
    task_entry["iterations_required"] = completion_report.iterations_required

    # Add checkpoint
    weaver_state["checkpoints"].append({
        "id": generate_uuid(),
        "description": f"Completed {task.name}",
        "timestamp": now(),
        "task_id": task.id,
        "resumable": True
    })

    save_weaver_state(weaver_state)

    # Git operations (if configured)
    if config.auto_commit:
        git_add(task.target_files)
        git_commit(f"feat({task.id}): {task.name}\n\n{completion_report.summary.what_was_done}")

    # Archive completion report
    archive_path = f".claude/skills/weaver/archive/completions/{task.id}.json"
    save_json(archive_path, completion_report)

    log(f"✓ Task {task.id} verified and recorded")
```

---

## Execution Modes

### Single Task
```bash
/ralph-run T3          # Run specific task
```

### Sequential Execution
```bash
/ralph-run --all       # Run all tasks in execution_plan order
```

### Phase Execution
```bash
/ralph-run --phase=2   # Run all tasks in phase 2
```

### Dry Run
```bash
/ralph-run T3 --dry-run    # Show what would happen without executing
```

### Resume from Failure
```bash
/ralph-run --resume        # Continue from last checkpoint
```

---

## Configuration

```json
{
  "ralph_executor": {
    "max_iterations": 5,
    "iteration_timeout_seconds": 300,
    "total_timeout_seconds": 1800,

    "reflection": {
      "min_confidence_threshold": 0.3,
      "loop_detection_sensitivity": "medium"
    },

    "gatekeeper": {
      "auto_approve": false,
      "security_scan_enabled": true,
      "require_revalidation": true
    },

    "git": {
      "auto_commit": false,
      "commit_message_template": "feat({task_id}): {task_name}"
    },

    "escalation": {
      "on_max_iterations": "pause",
      "on_blocked": "pause",
      "on_stuck": "pause",
      "notify_channel": null
    }
  }
}
```

---

## Error Handling

### Recoverable Errors

| Error | Recovery Action |
|-------|-----------------|
| Test flakiness detected | Retry validation 3x |
| Network timeout | Retry with backoff |
| File lock contention | Wait and retry |
| Partial write | Rollback and retry |

### Non-Recoverable Errors

| Error | Action |
|-------|--------|
| Max iterations exceeded | Escalate with history |
| Agent reports BLOCKED | Escalate with reason |
| Gatekeeper rejection | Escalate with diff |
| Security violation detected | HALT immediately |
| Out of scope modification | Rollback and escalate |

### Rollback Protocol

```python
def rollback_task(task):
    if task.rollback.strategy == "delete_created_files":
        for file in task.rollback.files_to_remove:
            if file_exists(file):
                delete_file(file)

    if task.rollback.strategy == "git_restore":
        for file in task.rollback.files_to_restore:
            git_checkout(file)

    # Update state
    weaver_state = load_weaver_state()
    task_entry = find_task(weaver_state, task.id)
    task_entry["status"] = "rolled_back"
    task_entry["rollback_at"] = now()
    save_weaver_state(weaver_state)
```

---

## Observability

### Execution Log Format

```
[2024-01-15 10:30:00] RALPH: Starting task T3 (Implement AuthContext)
[2024-01-15 10:30:01] RALPH: Assembling context (4 files, ~2500 tokens)
[2024-01-15 10:30:02] RALPH: Spawning isolated sub-agent
[2024-01-15 10:30:15] RALPH: Iteration 1/5 - ATTEMPT complete
[2024-01-15 10:30:18] RALPH: Iteration 1/5 - VERIFY: FAILED (exit code 1)
[2024-01-15 10:30:20] RALPH: Iteration 1/5 - REFLECT: Type mismatch identified
[2024-01-15 10:30:35] RALPH: Iteration 2/5 - ATTEMPT complete (fix applied)
[2024-01-15 10:30:38] RALPH: Iteration 2/5 - VERIFY: SUCCESS
[2024-01-15 10:30:38] RALPH: Sub-agent completed in 2 iterations
[2024-01-15 10:30:40] RALPH: Gatekeeper review starting
[2024-01-15 10:30:42] RALPH: ✓ DIFF_SCOPE: passed
[2024-01-15 10:30:43] RALPH: ✓ SECURITY_PATTERNS: passed
[2024-01-15 10:30:44] RALPH: ✓ CONSTRAINTS: passed
[2024-01-15 10:30:47] RALPH: ✓ REVALIDATION: passed
[2024-01-15 10:30:48] RALPH: ✓ SUCCESS_MARKERS: passed
[2024-01-15 10:30:48] RALPH: Gatekeeper verdict: APPROVED
[2024-01-15 10:30:49] RALPH: Task T3 marked as VERIFIED
```

### Metrics Collected

```json
{
  "task_id": "T3",
  "metrics": {
    "total_duration_seconds": 49,
    "iterations_required": 2,
    "context_tokens_used": 2500,
    "validation_runs": 2,
    "gatekeeper_checks_passed": 5,
    "files_created": 1,
    "files_modified": 1,
    "lines_added": 88,
    "lines_removed": 0
  }
}
```

---

## Integration Points

### With Lead Weaver
- Receives task assignments from `tasks.json`
- Reports completion status to `WEAVER_STATE.json`
- Escalates failures for human intervention

### With Verifier Agent
- May invoke Verifier for complex validation
- Receives detailed test reports

### With Git
- Optional auto-commit on task completion
- Rollback via git restore

---

## Command Reference

```bash
# Run specific task
/ralph-run T3

# Run all pending tasks
/ralph-run --all

# Run tasks in a specific phase
/ralph-run --phase=2

# Run with custom iteration limit
/ralph-run T3 --max-iterations=3

# Dry run (no changes)
/ralph-run T3 --dry-run

# Resume from checkpoint
/ralph-run --resume

# Run with verbose logging
/ralph-run T3 --verbose

# Force re-run completed task
/ralph-run T3 --force

# Run and auto-commit on success
/ralph-run T3 --commit
```
