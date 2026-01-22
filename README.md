# Claude Plugins Marketplace

Enterprise workflow automation plugins for Claude Code.

## Available Plugins

| Plugin | Description | Version |
|--------|-------------|---------|
| [workflow-weaver](./plugins/workflow-weaver) | Multi-agent workflow orchestration with task decomposition | 1.0.0 |

## Installation

```bash
# Add marketplace
/plugin marketplace add gotime4/claude-plugins

# Install plugin
/plugin install workflow-weaver@gotime4-claude-plugins

# Verify
/plugin
```

## Usage

```bash
/workflow-weaver:weave Add user authentication
/workflow-weaver:draft-prd Add payment integration
/workflow-weaver:ralph-run --all
```

## Security & Trust

**What this plugin does:**
- Orchestrates local file operations (read/write/edit)
- Runs shell commands you approve (tests, builds, git)
- Persists state in local JSON files

**What this plugin does NOT do:**
- No external API calls
- No network requests
- No telemetry or data collection
- No access beyond your project directory

**Safe defaults:**
- All file operations require your approval
- Git commits are never automatic unless you pass `--commit`
- No dependencies outside Claude Code's built-in tools

## Environment Support

| Environment | Status |
|-------------|--------|
| macOS | Supported |
| Linux | Supported |
| Windows | Supported (WSL recommended) |
| Offline/air-gapped | Fully supported |
| Corporate proxy | No external calls needed |

## Adding Plugins

1. Create folder in `plugins/`
2. Add `.claude-plugin/plugin.json`
3. Add commands in `commands/`
4. Update `marketplace.json`

## License

MIT
