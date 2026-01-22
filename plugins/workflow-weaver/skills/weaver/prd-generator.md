# PRD Generator Skill

## Command
```
/draft-prd <feature description>
```

## Overview

This skill generates a **WEAVER_BLUEPRINT.md**—a technical requirements document that goes beyond user stories to provide implementation-ready specifications. It operates in two mandatory phases: **Reflection** then **Drafting**.

---

## Phase 1: Reflection (MANDATORY)

**Purpose**: Understand the codebase's existing standards before writing any requirements. This prevents hallucinated patterns and ensures consistency.

### Reflection Protocol

Before writing a single line of the blueprint, you MUST:

#### 1.1 Project Discovery
```bash
# Identify project type and structure
ls -la
ls -la src/ 2>/dev/null || ls -la app/ 2>/dev/null || ls -la lib/ 2>/dev/null
cat package.json 2>/dev/null || cat Cargo.toml 2>/dev/null || cat pyproject.toml 2>/dev/null
```

#### 1.2 Pattern Detection (Context-Dependent)

Based on the feature request, search for relevant existing patterns:

**If UI/Component work:**
```bash
# Find component patterns
find . -name "*.tsx" -o -name "*.jsx" | head -20
grep -r "export.*function\|export.*const" src/components/ --include="*.tsx" | head -10

# Find styling patterns
grep -r "className\|styled\|css" src/components/ --include="*.tsx" | head -5
```

**If Testing involved:**
```bash
# Find test patterns
find . -name "*.test.*" -o -name "*.spec.*" | head -20
grep -r "data-testid\|data-test\|data-cy" . --include="*.tsx" --include="*.jsx" | head -10

# Find test utilities
ls -la **/test*/ 2>/dev/null
cat **/setupTests.* 2>/dev/null | head -30
```

**If API/Backend work:**
```bash
# Find API patterns
find . -name "*route*" -o -name "*controller*" -o -name "*handler*" | head -20
grep -r "app\.\(get\|post\|put\|delete\)\|router\." . --include="*.ts" --include="*.js" | head -10

# Find middleware patterns
grep -r "middleware\|use\(" . --include="*.ts" | head -10
```

**If State Management involved:**
```bash
# Find state patterns
grep -r "createContext\|useContext\|Provider" . --include="*.tsx" --include="*.ts" | head -10
grep -r "createStore\|useSelector\|useDispatch\|atom\|selector" . --include="*.ts" | head -10
```

**If Database/Models involved:**
```bash
# Find schema patterns
find . -name "*schema*" -o -name "*model*" -o -name "*migration*" | head -20
cat prisma/schema.prisma 2>/dev/null | head -50
grep -r "Schema\|Model\|Entity" . --include="*.ts" | head -10
```

#### 1.3 Read Exemplar Files

After identifying patterns, READ at least 2-3 files that exemplify the pattern:

```
Read the most relevant existing files to understand:
- Naming conventions
- Import patterns
- Code organization
- Error handling approach
- Comment/documentation style
```

#### 1.4 Document Findings

Create an internal reflection summary (not in final output):

```markdown
## Reflection Summary (Internal)

### Project Type
- Framework: [Next.js/React/Express/etc.]
- Language: [TypeScript/JavaScript/etc.]
- Package Manager: [npm/yarn/pnpm]

### Discovered Patterns
- Component Pattern: [functional with hooks / class-based / etc.]
- Styling: [Tailwind / CSS Modules / styled-components / etc.]
- Testing: [Jest + RTL / Vitest / Playwright / Cypress]
- State: [Context API / Redux / Zustand / etc.]
- API: [REST / GraphQL / tRPC]

### Naming Conventions
- Components: [PascalCase / kebab-case files]
- Tests: [*.test.ts / *.spec.ts]
- Test IDs: [data-testid="kebab-case" / data-test="camelCase"]

### Exemplar Files Referenced
- [file1]: [what pattern it demonstrates]
- [file2]: [what pattern it demonstrates]

### Unknowns Requiring Clarification
- [list anything that couldn't be determined from codebase]
```

---

## Phase 2: Drafting

**Purpose**: Generate a comprehensive WEAVER_BLUEPRINT.md based on reflection findings.

### Anti-Hallucination Rules

Before including ANY technical detail, verify:

| Detail Type | Verification Method | If Not Found |
|-------------|--------------------|--------------|
| Library/package | Check package.json/lock file | ASK USER |
| File path | Verify file exists with ls/find | ASK USER |
| Function/component name | Grep for exact name | ASK USER |
| Test selector pattern | Find existing test IDs | ASK USER |
| API endpoint pattern | Find existing routes | ASK USER |
| Environment variable | Check .env.example or config | ASK USER |

**CRITICAL**: Never guess:
- Library versions (check package.json)
- CSS class names (check actual files)
- Test ID naming conventions (find existing examples)
- API response shapes (find type definitions)
- Database column names (check schema)

If you cannot verify something exists in the codebase, you MUST ask:

```markdown
## Clarification Needed

Before I can complete this blueprint, I need clarification on:

1. **[Topic]**: I couldn't find [X] in the codebase.
   - What I searched: [commands run]
   - Options: [A] or [B] or [describe your preferred approach]

2. **[Topic]**: The existing pattern for [X] is unclear.
   - I found: [what was found]
   - Please confirm: [specific question]
```

---

## WEAVER_BLUEPRINT.md Template

```markdown
# WEAVER_BLUEPRINT: [Feature Name]

> Generated: [timestamp]
> Status: DRAFT | APPROVED
> Thread ID: [for linking to WEAVER_STATE]

---

## 1. Executive Summary

**Feature**: [one-line description]
**Requester**: [who asked for this]
**Priority**: [P0/P1/P2/P3]

### Business Context
[2-3 sentences on why this matters]

### Success Definition
[What does "done" look like from a user perspective?]

---

## 2. Existing Standards (From Reflection)

### Relevant Patterns Discovered

| Pattern | Location | Must Follow |
|---------|----------|-------------|
| [e.g., Component structure] | [e.g., src/components/Button.tsx] | [Yes/Recommended] |
| [e.g., Test ID format] | [e.g., src/components/*.test.tsx] | [Yes] |
| [e.g., API error handling] | [e.g., src/lib/api.ts] | [Yes] |

### Constraints From Codebase

1. **[Constraint Name]**: [Description]
   - Source: [Where this was discovered]
   - Example: `[code snippet]`

2. **[Constraint Name]**: [Description]
   - Source: [Where this was discovered]
   - Example: `[code snippet]`

---

## 3. Technical Implementation Spec

### Task 1: [Task Name]

#### Description
[What this task accomplishes]

#### Target Files

| File | Action | Reason |
|------|--------|--------|
| `src/components/NewFeature.tsx` | CREATE | Main component |
| `src/components/NewFeature.test.tsx` | CREATE | Unit tests |
| `src/hooks/useFeature.ts` | MODIFY | Add new hook export |

#### Implementation Details

```typescript
// Pseudo-code or interface showing expected shape
interface ExpectedImplementation {
  // Based on existing patterns from [exemplar file]
}
```

#### Constraints

- [ ] Must use existing `[PatternName]` from `[file]`
- [ ] Must follow naming convention: `[convention]`
- [ ] Must not modify: `[protected files]`
- [ ] Error handling must match: `[existing pattern location]`

#### Success Marker

```bash
# Command to verify completion
[exact command]

# Expected output (or pattern to match)
[expected output]
```

#### Dependencies
- Blocked by: [other task IDs if any]
- Blocks: [other task IDs if any]

---

### Task 2: [Task Name]

[Same structure as Task 1]

---

## 4. Test Strategy

### Unit Tests Required

| Test File | Coverage Target | Pattern Reference |
|-----------|-----------------|-------------------|
| `[file]` | [what to test] | `[existing test to follow]` |

### Integration Tests Required

| Test File | Scenario | Pattern Reference |
|-----------|----------|-------------------|
| `[file]` | [scenario] | `[existing test to follow]` |

### Manual Verification Checklist

- [ ] [Manual check 1]
- [ ] [Manual check 2]

---

## 5. Rollout Considerations

### Feature Flag
- Flag name: `[name]` (if applicable)
- Default: `[enabled/disabled]`

### Migration Required
- [ ] Database migration needed
- [ ] Data backfill needed
- [ ] Configuration change needed

### Rollback Plan
[How to revert if issues occur]

---

## 6. Open Questions

> Items that need resolution before implementation

| # | Question | Options | Recommendation | Status |
|---|----------|---------|----------------|--------|
| 1 | [Question] | A, B, C | [Your rec] | OPEN |
| 2 | [Question] | A, B | [Your rec] | RESOLVED: [answer] |

---

## 7. Approval

- [ ] Technical Lead approval
- [ ] Product approval (if applicable)
- [ ] Security review (if applicable)

**Approved by**: _______________
**Date**: _______________

---

## Appendix: Reflection Evidence

### Commands Run
```bash
[list of discovery commands and key findings]
```

### Files Referenced
- `[file1]`: [why referenced]
- `[file2]`: [why referenced]
```

---

## Workflow Integration

### Triggering from Lead Weaver

When `/draft-prd` is invoked:

1. Lead Weaver delegates to PRD Generator skill
2. PRD Generator executes Reflection Phase
3. If unknowns found → Return clarification questions (STOP)
4. If reflection complete → Generate WEAVER_BLUEPRINT.md
5. Return blueprint to Lead Weaver for review
6. On approval → Lead Weaver decomposes into WEAVER_STATE tasks

### Output Location

```
./WEAVER_BLUEPRINT.md           # Active blueprint
./.claude/skills/weaver/blueprints/  # Archived blueprints
```

### State Handoff

After blueprint approval, the PRD Generator provides structured handoff:

```json
{
  "blueprint_path": "./WEAVER_BLUEPRINT.md",
  "task_count": 5,
  "tasks_summary": [
    {"id": "T1", "name": "...", "files": ["..."], "estimated_complexity": "low|medium|high"},
    {"id": "T2", "name": "...", "files": ["..."], "estimated_complexity": "low|medium|high"}
  ],
  "open_questions": 0,
  "ready_for_implementation": true
}
```

---

## Anti-Pattern Examples

### BAD: Hallucinated Pattern
```markdown
#### Constraints
- Use the `data-cy` attribute for test selectors  # ❌ Never verified this exists
```

### GOOD: Verified Pattern
```markdown
#### Constraints
- Use `data-testid` attribute for test selectors
  - Source: Found in `src/components/Button.test.tsx:15`
  - Convention: kebab-case (e.g., `data-testid="submit-button"`)
```

### BAD: Assumed File Structure
```markdown
#### Target Files
- `src/features/auth/Login.tsx`  # ❌ Never verified this path exists
```

### GOOD: Verified File Structure
```markdown
#### Target Files
- `src/components/auth/LoginForm.tsx`
  - Verified: `ls src/components/auth/` shows existing auth components
  - Pattern: Components in flat structure, not nested features/
```

### BAD: Guessed Library Usage
```markdown
Use `axios.post()` for API calls  # ❌ Never checked what HTTP client is used
```

### GOOD: Discovered Library Usage
```markdown
Use `fetch()` wrapper from `src/lib/api.ts`
  - Source: `grep -r "fetch\|axios" src/` shows custom fetch wrapper
  - Pattern: All API calls go through `apiClient.request()`
```

---

## Command Variants

```bash
/draft-prd <description>              # Full reflection + draft
/draft-prd --skip-reflection <desc>   # Skip reflection (for follow-up PRDs in same domain)
/draft-prd --reflect-only <desc>      # Only run reflection, output findings
/draft-prd --from-issue <issue-url>   # Pull requirements from GitHub issue
```
