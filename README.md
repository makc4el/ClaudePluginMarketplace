# Claude Marketplace

Max's personal Claude Code plugin marketplace.

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
