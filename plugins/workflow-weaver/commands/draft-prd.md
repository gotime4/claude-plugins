# /draft-prd - Generate Technical Blueprint

```bash
/draft-prd <feature>
/draft-prd --from-plan <path>
/draft-prd --from-issue <github-url>
```

## Options

| Flag | Description |
|------|-------------|
| `--from-plan` | Use existing plan as input |
| `--from-issue` | Pull from GitHub issue |
| `--reflect-only` | Run reflection without drafting |
| `--output` | Custom output path |

## Phases

### 1. Reflection (mandatory)
- Discover project structure (framework, language)
- Detect patterns (components, tests, API, state)
- Read 2-3 exemplar files
- Identify unknowns → ask, don't guess

### 2. Drafting
Generate `WEAVER_BLUEPRINT.md` with:
- Target files (verified paths)
- Constraints (with source refs)
- Success markers (exact commands)
- Test strategy

## Anti-Hallucination

STOP and ask if you cannot verify:
- [ ] Library versions (check package.json)
- [ ] File paths (verify exist)
- [ ] Test patterns (find existing)
- [ ] API shapes (find types)

See `skills/weaver/prd-generator.md` for templates.
