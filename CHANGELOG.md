# Changelog

All notable changes to this project will be documented in this file.

## [1.0.0] - 2025-01-21

### Added
- Initial release of Workflow-Weaver plugin
- `/weave` - Start workflow threads from requirements or plan files
- `/draft-prd` - Generate technical blueprints with reflection phase
- `/decompose` - Transform blueprints into atomic task contracts
- `/ralph-run` - Execute tasks via attempt-verify-reflect loops
- `/weave-status`, `/weave-resume`, `/weave-abort` - Thread management
- Multi-agent architecture (Analyst, Architect, Implementer, Verifier)
- Anti-hallucination safeguards in PRD generation
- Context isolation for sub-agents (~50k token limit)
- `--max-loops` flag for validation retry control
- `--model` flag for sub-agent model selection
- `--from-plan` flag for /plan integration
- State persistence via WEAVER_STATE.json
- Gatekeeper review before task completion
