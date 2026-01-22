# /weave - Start a New Workflow Thread

## Invocation

```bash
# From a short description
/weave <requirement>

# From a plan file (created via /plan or manually)
/weave --from-plan <path-to-plan.md>

# From pasted plan content
/weave --from-plan <<EOF
<paste your plan here>
EOF

# From the default plan location
/weave --from-plan ./PLAN.md
```

## Input Sources

### 1. Direct Requirement (Simple)
```bash
/weave Add user authentication with JWT tokens
```
Best for: Simple, well-understood features

### 2. From /plan Output (Recommended for Complex Features)

First, use Claude Code's built-in plan mode:
```bash
/plan Add user authentication with JWT tokens and refresh token rotation
```

This creates a detailed plan. Then feed it to weave:
```bash
/weave --from-plan ./PLAN.md
```

### 3. From Pasted Content

```bash
/weave --from-plan <<EOF
## Feature: User Authentication

### Requirements
- JWT-based authentication
- Refresh token rotation
- Session management

### Acceptance Criteria
- Users can log in with email/password
- Tokens expire after 15 minutes
- Refresh tokens rotate on use
EOF
```

### 4. From GitHub Issue

```bash
/weave --from-issue https://github.com/org/repo/issues/123
```

## Behavior

When invoked, the Lead Weaver will:

1. **Parse Input**
   - If `--from-plan`: Read and parse the plan document
   - If `--from-issue`: Fetch issue content via `gh` CLI
   - Otherwise: Use the requirement string directly

2. **Initialize State**
   - Generate new `session_id` and `thread.id`
   - Store the requirement/plan in `thread.requirement`
   - If from plan: Extract structured sections (requirements, acceptance criteria, etc.)
   - Set `thread.status` to "analyzing"
   - Create initial checkpoint

3. **Analyze Requirement**
   - Assess complexity (files affected, architectural decisions, dependencies)
   - Determine risk level
   - Identify ambiguities requiring clarification
   - If from plan: Skip clarification for items already specified

4. **Decompose into Tasks**
   - Create discrete, bounded tasks
   - Assign appropriate agent types
   - Define input context and success criteria for each

5. **Begin Orchestration**
   - Dispatch first task (typically analysis)
   - Wait for completion and verification
   - Proceed through task chain with checkpoints

## Plan Document Format

When using `--from-plan`, the document can be:

### Markdown Format (Flexible)
```markdown
# Feature: [Title]

## Overview
[Description of what we're building]

## Requirements
- [Requirement 1]
- [Requirement 2]

## Acceptance Criteria
- [ ] [Criterion 1]
- [ ] [Criterion 2]

## Technical Notes
[Any implementation hints or constraints]

## Out of Scope
[What we're NOT doing]
```

### Claude /plan Output (Auto-Detected)
The output from `/plan` mode is automatically recognized and parsed.

### Structured YAML Front Matter (Advanced)
```markdown
---
title: User Authentication
priority: high
scope:
  - src/auth/**
  - src/middleware/**
constraints:
  - Must use existing UserContext
  - No new dependencies without approval
---

## Requirements
...
```

## Example Workflows

### Workflow A: Quick Feature
```bash
/weave Add a logout button to the navbar
```

### Workflow B: Complex Feature with Planning
```bash
# Step 1: Plan with Claude
/plan Implement full user authentication with JWT, refresh tokens,
      password reset, and email verification

# Step 2: Review and refine the plan (Claude will iterate with you)

# Step 3: Execute the approved plan
/weave --from-plan ./PLAN.md
```

### Workflow C: From Existing Documentation
```bash
# If you have a PRD or spec document
/weave --from-plan ./docs/specs/auth-feature.md
```

## Options

| Option | Description | Default |
|--------|-------------|---------|
| `--from-plan <path>` | Use a plan document as input | - |
| `--from-issue <url>` | Pull requirements from GitHub issue | - |
| `--model=<model>` | Model for sub-agents: `inherit`, `haiku`, `sonnet`, `opus` | `inherit` |
| `--max-loops=N` | Max validation retries per task | `5` |
| `--fail-fast` | Stop on first validation failure (`--max-loops=1`) | - |
| `--dry-run` | Show decomposition without starting | - |
| `--priority=<level>` | Set priority (low/medium/high/critical) | - |
| `--scope=<files>` | Pre-constrain file scope | - |
| `--skip-analysis` | Skip analysis phase (use with detailed plans) | - |

## Examples with Model and Loop Control

```bash
# Default: inherit model, 5 retries per task
/weave Add user authentication

# Use Sonnet for sub-agents (balanced)
/weave --from-plan ./PLAN.md --model=sonnet

# Use Haiku for speed/cost (simpler tasks)
/weave --from-plan ./PLAN.md --model=haiku

# Use Opus for complex features
/weave --from-plan ./PLAN.md --model=opus

# Conservative: fail fast, review errors manually
/weave --from-plan ./PLAN.md --max-loops=2

# Combined: Haiku with limited retries
/weave --from-plan ./PLAN.md --model=haiku --max-loops=2
```

## Notes

- Only one thread can be active at a time
- Use `/weave-status` to check progress
- Use `/weave-abort` to cancel if needed
- Threads auto-checkpoint; use `/weave-resume` after interruption
- Plans from `/plan` mode preserve the analysis already done
- `--max-loops` applies to all tasks in the thread
