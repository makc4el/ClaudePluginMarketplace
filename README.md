# Claude Marketplace

A personal Claude Code plugin marketplace.

## Setup

Register this repo as a local marketplace once, then install any plugin from it:

```bash
# 1. Add the marketplace (run once, absolute path required)
claude plugin marketplace add /path/to/claude-marketplace

# 2. Install a plugin
claude plugin install <plugin-name>@local-plugins
```

To verify it was added:

```bash
claude plugin marketplace list
```

To update all plugins after pulling new changes:

```bash
claude plugin marketplace update local-plugins
```

## Plugins

| Plugin | Description |
|--------|-------------|
| `arch-sync` | Multi-repo system architecture visualizer |

## Adding a New Plugin

1. Create `plugins/<plugin-name>/` with:
   - `.claude-plugin/plugin.json` — metadata
   - `commands/` — slash commands (optional)
   - `skills/` — skills (optional)
   - `agents/` — agents (optional)
   - `README.md`

2. Register in `.claude-plugin/marketplace.json` under `plugins`:
   ```json
   {
     "name": "<plugin-name>",
     "description": "...",
     "version": "1.0.0",
     "author": { "name": "Max Odarchenko" },
     "source": "./plugins/<plugin-name>",
     "category": "development"
   }
   ```

3. Install: `claude plugin install <plugin-name>@local-plugins`
