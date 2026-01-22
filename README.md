# Claude Plugins Marketplace

A collection of Claude Code plugins for enterprise workflow automation.

## Available Plugins

| Plugin | Description | Version |
|--------|-------------|---------|
| [workflow-weaver](./plugins/workflow-weaver) | Enterprise-grade AI workflow orchestration with multi-agent task decomposition | 1.0.0 |

## Installation

### 1. Add the Marketplace

```bash
/plugin marketplace add gotime4/claude-plugins
```

Replace `gotime4` with your GitHub username after pushing this repo.

### 2. Install a Plugin

```bash
/plugin install workflow-weaver@gotime4-claude-plugins
```

### 3. Verify Installation

```bash
/plugin
```

## Usage

After installation, plugin commands are namespaced:

```bash
# Start a workflow
/workflow-weaver:weave Add user authentication

# Generate a PRD
/workflow-weaver:draft-prd Add payment integration

# Run tasks
/workflow-weaver:ralph-run --all
```

## Publishing to GitHub

1. Create a new repository on GitHub
2. Push this folder:

```bash
cd claude-plugins
git init
git add .
git commit -m "Initial commit: Workflow-Weaver plugin"
git remote add origin https://github.com/gotime4/claude-plugins.git
git push -u origin main
```

3. Users can then install with:

```bash
/plugin marketplace add gotime4/claude-plugins
/plugin install workflow-weaver@gotime4-claude-plugins
```

## Adding More Plugins

To add another plugin:

1. Create a new folder in `plugins/`
2. Add `.claude-plugin/plugin.json` manifest
3. Add commands in `commands/`
4. Add skills in `skills/`
5. Update `marketplace.json` with the new plugin entry

## License

MIT
