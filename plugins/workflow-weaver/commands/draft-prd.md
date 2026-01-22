# /draft-prd - Generate Technical Blueprint

## Invocation

```bash
# From a short description
/draft-prd <feature description>

# From a plan file (created via /plan or manually)
/draft-prd --from-plan <path-to-plan.md>

# From pasted plan content
/draft-prd --from-plan <<EOF
<paste your plan here>
EOF

# From GitHub issue
/draft-prd --from-issue https://github.com/org/repo/issues/123
```

## Input Sources

### 1. Direct Description (Simple)
```bash
/draft-prd Add user profile page with avatar upload
```
Best for: Simple features where you want full reflection

### 2. From /plan Output (Recommended)

First, use Claude Code's built-in plan mode to think through the feature:
```bash
/plan Add user authentication with JWT tokens, refresh token rotation,
      and protected routes
```

Claude will analyze the codebase and create a detailed plan. Then convert it to a technical blueprint:
```bash
/draft-prd --from-plan ./PLAN.md
```

**Why use /plan first?**
- `/plan` mode focuses on *what* to build and *why*
- `/draft-prd` adds *how* with technical specs, file targets, and validation commands
- The combination gives you both strategic thinking and implementation details

### 3. From Pasted Content

```bash
/draft-prd --from-plan <<EOF
## Feature: User Authentication

### Requirements
- JWT-based authentication with 15-minute expiry
- Refresh token rotation for security
- Protected route middleware

### User Stories
- As a user, I can log in with email/password
- As a user, I stay logged in across browser sessions
- As a user, I am automatically logged out after inactivity

### Technical Constraints
- Must integrate with existing UserContext
- Use httpOnly cookies for refresh tokens
EOF
```

### 4. From GitHub Issue

```bash
/draft-prd --from-issue https://github.com/org/repo/issues/123
```

## Behavior

The PRD Generator operates in two mandatory phases:

### Phase 1: Reflection (Cannot Skip)

Before writing ANY requirements, the system will:

1. **Parse Input**
   - If `--from-plan`: Extract requirements, user stories, constraints from the plan
   - If `--from-issue`: Fetch and parse issue content
   - Otherwise: Use description as starting point

2. **Discover Project Structure**
   - Identify framework, language, package manager
   - Map directory structure

3. **Detect Relevant Patterns** (based on feature type)
   - UI work → Find component patterns, styling conventions
   - Testing → Find test framework, selector conventions, test utilities
   - API work → Find routing patterns, middleware, error handling
   - State → Find state management approach
   - Database → Find schema patterns, ORM usage

4. **Read Exemplar Files**
   - Read 2-3 files that demonstrate the patterns
   - Extract naming conventions, import patterns, code organization

5. **Cross-Reference with Plan**
   - If plan specifies technical constraints → Verify they match codebase
   - If plan mentions specific files → Verify they exist
   - If plan assumes patterns → Validate against discovered patterns

6. **Identify Unknowns**
   - If any pattern cannot be determined from codebase → Queue for clarification
   - If plan contradicts codebase patterns → Flag for resolution

### Phase 2: Drafting

Generate `WEAVER_BLUEPRINT.md` containing:

- **Executive Summary**: What, why, and success definition
- **Source Reference**: Link to original plan/issue if applicable
- **Existing Standards**: Patterns discovered during reflection
- **Technical Implementation Spec**: For each task:
  - Target Files (verified to exist or explicitly marked CREATE)
  - Constraints (with source references)
  - Success Markers (exact commands and expected output)
- **Test Strategy**: Unit, integration, and manual verification
- **Open Questions**: Anything requiring human decision
- **Plan Alignment**: How the blueprint maps to the original plan (if from plan)

## Anti-Hallucination Safeguards

The generator will STOP and ask for clarification rather than guess:

- Library versions or packages not in dependency files
- File paths that don't exist
- Test selector patterns not found in existing tests
- API shapes not defined in types
- Environment variables not in .env.example
- Plan assumptions that don't match discovered patterns

## Example Workflows

### Workflow A: Quick PRD
```bash
/draft-prd Add dark mode toggle to settings
```

### Workflow B: Plan-First (Recommended for Complex Features)
```bash
# Step 1: Strategic planning with Claude
/plan Implement a real-time notification system with WebSocket support,
      push notifications, and in-app notification center

# Step 2: Claude helps you think through:
#   - Architecture decisions
#   - User experience flow
#   - Edge cases and error handling
#   - Phasing/rollout strategy

# Step 3: Convert approved plan to technical blueprint
/draft-prd --from-plan ./PLAN.md

# Step 4: Review WEAVER_BLUEPRINT.md
# Step 5: Decompose and execute
/decompose
/ralph-run --all
```

### Workflow C: From Existing Spec
```bash
# If PM gave you a requirements doc
/draft-prd --from-plan ./docs/requirements/notifications-prd.md
```

## Plan Document Format

When using `--from-plan`, the following sections are recognized:

```markdown
# Feature: [Title]

## Overview / Summary
[High-level description]

## Requirements / Goals
- [Requirement 1]
- [Requirement 2]

## User Stories (Optional)
- As a [user], I can [action] so that [benefit]

## Acceptance Criteria
- [ ] [Testable criterion]

## Technical Constraints (Optional)
- [Constraint that must be followed]

## Out of Scope (Optional)
- [What we're explicitly NOT doing]

## Open Questions (Optional)
- [Unresolved decisions]

## Implementation Notes (Optional)
- [Any technical hints or preferences]
```

## Options

| Option | Description |
|--------|-------------|
| `--from-plan <path>` | Use a plan document as input |
| `--from-issue <url>` | Pull requirements from GitHub issue |
| `--reflect-only` | Only run reflection, output findings without blueprint |
| `--skip-reflection` | Skip reflection (DANGEROUS: may hallucinate patterns) |
| `--output <path>` | Custom output path (default: ./WEAVER_BLUEPRINT.md) |

## Output

Primary: `./WEAVER_BLUEPRINT.md`
Archive: `.claude/skills/weaver/blueprints/<timestamp>-<feature>.md`

## Integration with Lead Weaver

After approval, the blueprint is handed to Lead Weaver which:
1. Parses tasks into `WEAVER_STATE.json`
2. Assigns appropriate sub-agents
3. Begins orchestrated implementation

## /plan vs /draft-prd

| Aspect | /plan | /draft-prd |
|--------|-------|------------|
| Focus | Strategy & design | Implementation specs |
| Output | Plan document | WEAVER_BLUEPRINT.md |
| Depth | What & why | How & where |
| Validation | Conceptual review | File paths, commands verified |
| Best for | Thinking through approach | Ready-to-execute specs |

**Use together**: `/plan` → think → `/draft-prd --from-plan` → execute

## Notes

- Reflection phase typically examines 10-30 files
- If >3 clarification questions arise, consider splitting the feature
- Blueprints are versioned; edits create new versions
- Plans from `/plan` mode accelerate the process by pre-answering many questions
