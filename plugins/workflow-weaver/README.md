# Workflow-Weaver

Enterprise-grade AI workflow orchestration system for Claude Code.

## Overview

Workflow-Weaver transforms Claude Code into a disciplined software engineering system with:

- **Multi-agent architecture** with strict context isolation
- **Anti-hallucination safeguards** that verify patterns before generating specs
- **Autonomous execution loops** that iterate until validation passes
- **Gatekeeper reviews** before any code is marked complete

## Installation

### Via Marketplace (Recommended)

```bash
# Add the marketplace
/plugin marketplace add gotime4/claude-plugins

# Install the plugin
/plugin install workflow-weaver@gotime4-claude-plugins
```

### Manual Installation

Copy the plugin folder to your project's `.claude/` directory.

## Commands

| Command | Description |
|---------|-------------|
| `/workflow-weaver:weave <requirement>` | Start a new workflow thread |
| `/workflow-weaver:weave-status` | Check current thread progress |
| `/workflow-weaver:weave-resume` | Resume an interrupted thread |
| `/workflow-weaver:weave-abort` | Cancel current thread |
| `/workflow-weaver:draft-prd <feature>` | Generate technical blueprint with reflection |
| `/workflow-weaver:decompose` | Transform blueprint into atomic tasks |
| `/workflow-weaver:ralph-run [task-id]` | Execute tasks via Ralph Wiggum Loop |

## Architecture

```
                    ┌─────────────────────┐
                    │    LEAD WEAVER      │
                    │   (Orchestrator)    │
                    └──────────┬──────────┘
                               │
         ┌─────────────────────┼─────────────────────┐
         │                     │                     │
         ▼                     ▼                     ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│  PRD Generator  │  │   Decomposer    │  │ Ralph Executor  │
│  (Reflection +  │  │  (Atomicity +   │  │ (Loop Manager)  │
│   Drafting)     │  │   Scoping)      │  │                 │
└─────────────────┘  └─────────────────┘  └────────┬────────┘
                                                   │
                              ┌────────────────────┼────────────────────┐
                              │                    │                    │
                              ▼                    ▼                    ▼
                     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
                     │   Analyst    │     │  Architect   │     │ Implementer  │
                     │   Agent      │     │   Agent      │     │    Agent     │
                     └──────────────┘     └──────────────┘     └──────────────┘
```

## Workflow

### Recommended: Plan-First Workflow

For complex features, use Claude's built-in `/plan` mode first:

```bash
# Step 1: Strategic planning (built-in Claude Code)
/plan Implement user authentication with JWT, refresh tokens, and protected routes

# Step 2: Claude helps you think through architecture, edge cases, etc.
#         Review and refine the plan interactively

# Step 3: Convert plan to technical blueprint
/workflow-weaver:draft-prd --from-plan ./PLAN.md

# Step 4: Decompose into atomic tasks
/workflow-weaver:decompose

# Step 5: Execute with autonomous agents
/workflow-weaver:ralph-run --all
```

### /plan vs /draft-prd

| Aspect | /plan | /draft-prd |
|--------|-------|------------|
| Focus | Strategy & design | Implementation specs |
| Output | PLAN.md | WEAVER_BLUEPRINT.md |
| Depth | What & why | How & where (file paths, commands) |
| Validation | Conceptual | File paths verified, patterns checked |

**Use together**: `/plan` → think → `/draft-prd --from-plan` → execute

---

### Quick Workflow (Simple Features)

```bash
# Skip /plan for simple, well-understood features
/workflow-weaver:draft-prd Add logout button to navbar
/workflow-weaver:decompose
/workflow-weaver:ralph-run --all
```

---

### Step-by-Step Details

#### 1. Draft PRD (Reflection Phase)

```bash
# From description
/workflow-weaver:draft-prd Add user authentication with JWT

# From plan file
/workflow-weaver:draft-prd --from-plan ./PLAN.md

# From GitHub issue
/workflow-weaver:draft-prd --from-issue https://github.com/org/repo/issues/123
```

The PRD Generator:
1. **Reflects** - Searches codebase for existing patterns (test conventions, component structure, API patterns)
2. **Cross-references** - Validates plan assumptions against actual codebase
3. **Asks** - If patterns can't be found, asks for clarification instead of guessing
4. **Drafts** - Generates `WEAVER_BLUEPRINT.md` with verified technical specs

#### 2. Decompose into Tasks

```bash
/workflow-weaver:decompose
```

Transforms the blueprint into `tasks.json` with:
- **Atomicity**: Max 5 files, 300 lines per task
- **Context Scoping**: Each task gets only the files it needs (~50k tokens max)
- **Validation**: Every task has a `validation_command`

#### 3. Execute via Ralph Loop

```bash
/workflow-weaver:ralph-run --all
```

The Ralph Wiggum Loop ("I'm learnding!"):
```
ATTEMPT → VERIFY → REFLECT → ITERATE
   │                            │
   └────────────────────────────┘
         (max 5 iterations)
```

#### 4. Gatekeeper Review

Before marking any task complete:
- Diff is within declared scope
- No security violations
- All constraints followed
- Validation passes
- Success markers verified

## Key Features

### Anti-Hallucination

The system will **STOP and ask** rather than guess:
- Library versions not in package.json
- File paths that don't exist
- Test selector patterns not in codebase
- API shapes not defined in types

### Process Isolation

Sub-agents are stateless and receive only:
- Task specification
- Required context files (capped at ~50k tokens)
- No access to other tasks or system state

### State Persistence

`WEAVER_STATE.json` tracks:
- Thread status and checkpoints
- Task completion status
- Boundary violations
- Session recovery data

## Files

```
skills/weaver/
├── SKILL.md              # Lead Weaver persona
├── WEAVER_STATE.json     # State persistence
├── BOUNDARIES.json       # Governance rules
├── prd-generator.md      # PRD skill
├── ralph-executor.md     # Execution skill
├── ralph-config.json     # Executor config
├── task-schema.json      # tasks.json schema
└── agents/
    ├── analyst.md        # Requirements agent
    ├── architect.md      # Design agent
    ├── implementer.md    # Coding agent
    └── verifier.md       # Testing agent
```

## Configuration

Edit `skills/weaver/ralph-config.json` to customize:
- Max iterations (default: 5)
- Timeout settings
- Auto-commit behavior
- Escalation policies

## License

MIT
