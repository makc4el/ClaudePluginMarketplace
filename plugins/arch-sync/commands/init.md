---
description: Creates arch-sync.json by detecting sibling repos and asking which ones to include in system context
argument-hint: "[--path <directory>] [--yes]"
allowed-tools: Read, Write, Bash, Glob
---

# arch-sync: init

Initialize `arch-sync.json` for this repo by discovering sibling repos and selecting which to include.

## What this command does

Interactively creates an `arch-sync.json` config file in the current directory that lists all sibling repos to include when running `/arch-sync:sync`.

## Step-by-step instructions

### Step 1: Detect the current repo

1. Get the current working directory (use Bash `pwd`)
2. Determine the repo name: use the directory name (e.g., `langgraph` from `/Users/max/Projects/mcp/langgraph`)
3. Get the parent directory path (e.g., `/Users/max/Projects/mcp`)

### Step 2: Check for existing config

If `arch-sync.json` already exists in the current directory:
- Tell the user it already exists
- Show its current content
- Ask: "Would you like to overwrite it, or update/add repos to it?"
- If updating: load the existing config and merge new repos into it

### Step 3: Detect sibling directories

Use Bash to list sibling directories:
```bash
ls -d <parent-dir>/*/
```

Filter out:
- Hidden directories (starting with `.`)
- The current repo's own directory
- Non-directory entries

Show the user the list of discovered sibling directories.

If `--path <directory>` was provided, scan that directory instead of the auto-detected parent.

### Step 4: Ask which repos to include

Show the discovered sibling directories and ask the user to select which ones to include. Present them as a numbered list. Ask the user to enter the numbers or names of repos to include (or "all").

For each selected repo, show what documentation files already exist in it:
- Check for `CLAUDE.md`, `INTEGRATION.md`, `README.md`
- Show which are present (✅) and which are missing (⚠️)

### Step 5: Configure files to scan per repo

For each selected repo, determine which files to include. The defaults are `["CLAUDE.md", "INTEGRATION.md", "README.md"]`.

If `--yes` flag was provided, use defaults for all repos without asking.

Otherwise, for each repo that has non-standard files (e.g., missing INTEGRATION.md), ask: "Which files should be synced from <repo>? (defaults: CLAUDE.md, INTEGRATION.md, README.md)"

### Step 6: Write arch-sync.json

Create `arch-sync.json` with this format:

```json
{
  "currentRepo": {
    "name": "<detected-repo-name>",
    "description": "<ask user for one sentence, or use CLAUDE.md first line if exists>"
  },
  "repos": [
    {
      "name": "<repo-dir-name>",
      "path": "<relative-path-from-current-repo>",
      "files": ["CLAUDE.md", "INTEGRATION.md", "README.md"]
    }
  ]
}
```

Use relative paths (e.g., `"../mcp"` not absolute paths).

Only include `"files"` in a repo entry if they differ from the defaults. Use defaults implicitly when possible.

### Step 7: Offer to run sync

After creating the config, ask: "arch-sync.json created with N repos. Would you like to run /arch-sync:sync now to generate ARCHITECTURE.md?"

If yes, proceed with the full sync as described in the sync command.

## Example output

```
🔍 Detected parent directory: /Users/max/Projects/mcp
📦 Current repo: langgraph

Sibling directories found:
  1. mcp          (✅ CLAUDE.md  ✅ INTEGRATION.md  ⚠️  README.md missing)
  2. salesforce-ai-automator  (✅ CLAUDE.md  ✅ README.md  ⚠️  INTEGRATION.md missing)
  3. ai-agent-context-sync   (✅ CLAUDE.md  ✅ README.md  ⚠️  INTEGRATION.md missing)

Which repos should be included? (Enter numbers, names, or "all"):
> 1 2 3

✅ arch-sync.json created with 3 repos.
Run /arch-sync:sync to generate ARCHITECTURE.md.
```

## Argument handling

- `--path <directory>`: Scan a specific directory for sibling repos instead of auto-detecting the parent
- `--yes`: Accept all defaults (include all detected repos, use default file list) without interactive prompts

## Error handling

- Parent directory not readable → tell the user and ask to specify `--path` manually
- No sibling directories found → tell the user and ask them to specify repo paths manually
- Conflicting repo names in config → warn and ask which to keep
