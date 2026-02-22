# arch-sync

**Multi-repo system architecture visualizer for Claude Code.**

When working on a multi-platform project (multiple repos in different languages/frameworks), Claude only knows the context of the repo you're currently in. `arch-sync` solves this by:

1. Reading `CLAUDE.md`, `INTEGRATION.md`, and `README.md` from each sibling repo
2. Caching them locally in `.claude/system-context/<repo-name>/`
3. Generating `ARCHITECTURE.md` — a unified system view from *this* repo's perspective

Once generated, `ARCHITECTURE.md` gives Claude full context about:
- What depends on this repo
- What this repo depends on
- End-to-end data flows through the system
- Shared contracts (auth, schemas, protocols, env vars)

## Installation

```bash
# Install globally (available in all projects)
cc plugin install ~/arch-sync

# Or use directly without installing
cc --plugin-dir ~/arch-sync
```

## Quick Start

### 1. Initialize config (first time)

```
/arch-sync:init
```

Claude will detect sibling repos and help you create `arch-sync.json`.

### 2. Sync and generate ARCHITECTURE.md

```
/arch-sync:sync
```

Claude reads all configured repos, copies their docs locally, and writes `ARCHITECTURE.md`.

### 3. Re-sync after changes

```
/arch-sync:sync                    # Re-sync everything
/arch-sync:sync --repo mcp         # Re-sync one repo only
/arch-sync:sync --no-copy          # Regenerate ARCHITECTURE.md from cached context
```

## Config File (arch-sync.json)

Created by `/arch-sync:init`. Place at project root.

```json
{
  "currentRepo": {
    "name": "langgraph",
    "description": "Multi-agent LangGraph system for Salesforce automation"
  },
  "repos": [
    {
      "name": "mcp",
      "path": "../mcp"
    },
    {
      "name": "salesforce-ai-automator",
      "path": "../salesforce-ai-automator",
      "files": ["CLAUDE.md", "README.md"]
    },
    {
      "name": "ai-agent-context-sync",
      "path": "../ai_agent_context_sync"
    }
  ]
}
```

**Fields:**
- `repos[].name` — Display name for this repo (used as folder name in `.claude/system-context/`)
- `repos[].path` — Path to the repo, relative to current project root
- `repos[].files` — *(optional)* Which files to scan. Defaults to `["CLAUDE.md", "INTEGRATION.md", "README.md"]`

## What Gets Generated

### `.claude/system-context/` (local cache)

```
.claude/
└── system-context/
    ├── mcp/
    │   ├── CLAUDE.md
    │   ├── INTEGRATION.md
    │   └── README.md
    ├── salesforce-ai-automator/
    │   ├── CLAUDE.md
    │   └── README.md
    └── ai-agent-context-sync/
        └── CLAUDE.md
```

Commit this folder to give your team (and CI) full system context without needing all repos checked out.

### `ARCHITECTURE.md` (project root)

A comprehensive architecture document covering:
- System components table
- Inbound dependencies (who calls/uses this repo)
- Outbound dependencies (what this repo calls/uses)
- Data flows (2-5 key end-to-end flows involving this repo)
- Integration contracts (auth patterns, shared schemas, transport protocols, shared env vars)

Commit this file — it's designed to be the first thing Claude reads when helping with this repo.

## Commands

| Command | Description |
|---|---|
| `/arch-sync:init` | First-time setup: detects sibling repos, creates arch-sync.json |
| `/arch-sync:sync` | Full sync: reads all repos, copies docs, regenerates ARCHITECTURE.md |
| `/arch-sync:sync --repo <name>` | Re-sync one repo, regenerate ARCHITECTURE.md |
| `/arch-sync:sync --no-copy` | Regenerate ARCHITECTURE.md from cached context only |
| `/arch-sync:sync --force` | Overwrite all cached files even if unchanged |

## Recommended Workflow

1. Add `/arch-sync:sync` to your project setup checklist
2. Re-run when any repo's CLAUDE.md or INTEGRATION.md changes significantly
3. Commit both `.claude/system-context/` and `ARCHITECTURE.md` so teammates and CI get full context
4. Reference `ARCHITECTURE.md` in your repo's own CLAUDE.md: "See ARCHITECTURE.md for full system context"
