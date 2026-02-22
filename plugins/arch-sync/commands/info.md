---
description: Lists all arch-sync commands and explains how to use the plugin to keep multi-repo system documentation in sync
allowed-tools: []
---

# arch-sync: info

Print a concise help reference for the arch-sync plugin.

## What to output

Print the following text verbatim (you may render it as markdown):

---

## arch-sync — Multi-Repo Architecture Sync

Keeps your multi-repo system documented and Claude-aware. Three commands, one workflow.

---

### Commands

#### `/arch-sync:describe` — Document a repo
Analyzes source files and generates `CLAUDE.md` (internal dev guide) and `INTEGRATION.md` (external integration reference) for the **current repo**.

Run this once in each repo that is part of your system — especially ones that are missing docs.

```
/arch-sync:describe                        # generate both files
/arch-sync:describe --claude-only          # only CLAUDE.md
/arch-sync:describe --integration-only    # only INTEGRATION.md
/arch-sync:describe --force               # overwrite existing files
/arch-sync:describe --context "TypeScript MCP server wrapping Salesforce"
```

**Produces:** `CLAUDE.md`, `INTEGRATION.md`

---

#### `/arch-sync:init` — Configure which repos to watch
Detects sibling repos and creates `arch-sync.json` — the config file that tells `/arch-sync:sync` which repos to pull docs from.

Run this once in the repo you want to be your "system view" repo (typically your main backend or orchestration layer).

```
/arch-sync:init             # interactive — pick repos from a list
/arch-sync:init --yes       # accept all detected repos with defaults
/arch-sync:init --path ../  # scan a custom directory for sibling repos
```

**Produces:** `arch-sync.json`

---

#### `/arch-sync:sync` — Pull docs and regenerate ARCHITECTURE.md
Reads all repos listed in `arch-sync.json`, copies their `CLAUDE.md` / `INTEGRATION.md` / `README.md` into `.claude/system-context/`, and generates `ARCHITECTURE.md` for the current repo.

Run this any time a sibling repo's docs change.

```
/arch-sync:sync                   # full sync all repos
/arch-sync:sync --repo mcp        # refresh one repo only
/arch-sync:sync --no-copy         # regenerate ARCHITECTURE.md from cached docs
/arch-sync:sync --force           # overwrite cached files even if unchanged
```

**Produces:** `.claude/system-context/<repo>/`, `ARCHITECTURE.md`

---

### Typical Workflow

```
# Step 1 — Document each repo (run inside each repo)
cd ../mcp && /arch-sync:describe
cd ../salesforce-ai-automator && /arch-sync:describe

# Step 2 — Configure the system view (run once in your main repo)
cd ../langgraph && /arch-sync:init

# Step 3 — Generate the architecture doc (run in your main repo)
/arch-sync:sync
```

After setup, the only command you'll use regularly is `/arch-sync:sync` — run it whenever something changes.

---

### What gets committed

| File | Commit? | Purpose |
|---|---|---|
| `arch-sync.json` | ✅ Yes | Keeps config consistent across team |
| `CLAUDE.md` | ✅ Yes | Claude reads this every session |
| `INTEGRATION.md` | ✅ Yes | Other repos' Claude sessions read this |
| `ARCHITECTURE.md` | ✅ Yes | Full system view, always up to date |
| `.claude/system-context/` | ⚠️ Optional | Cached copies of sibling docs — useful to commit so Claude has context without re-running sync |

---

Run `/arch-sync:info` any time you need a reminder.
