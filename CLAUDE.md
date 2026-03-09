# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A personal Claude Code plugin marketplace. It hosts plugins that can be installed into any Claude Code project via `claude plugin install <plugin-name>@local-plugins`.

The marketplace registry is `.claude-plugin/marketplace.json`. Each plugin lives under `plugins/<plugin-name>/`.

## Plugin Structure

Each plugin follows this layout:

```
plugins/<plugin-name>/
├── .claude-plugin/plugin.json   # plugin metadata (name, version, description, keywords)
├── commands/                    # slash commands — one .md file per command
├── skills/                      # skills — subdirectory per skill with SKILL.md
├── agents/                      # agents (optional)
└── README.md
```

**Commands** (`commands/<name>.md`) use YAML frontmatter:
- `description` — shown in command picker
- `argument-hint` — displayed as usage hint
- `allowed-tools` — list of tools the command may use

**Skills** (`skills/<name>/SKILL.md`) use YAML frontmatter:
- `name`, `description`, `version`
- The body is the full skill prompt — it instructs Claude how to execute the skill

## Adding a New Plugin

1. Create `plugins/<plugin-name>/` with the structure above
2. Add an entry to `.claude-plugin/marketplace.json` under `"plugins"`:
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
3. Install locally: `claude plugin install <plugin-name>@local-plugins`

## Current Plugins

### figma-responsive

Generates unified responsive React/Next.js components from multiple Figma frames (desktop + mobile, optionally tablet). Fetches each frame via the local Figma MCP server, inspects the target project's Tailwind config and component patterns, then produces the full design-system file set with Tailwind breakpoints, `cva` variants, and shared hooks/utils.

**Commands:**
- `/figma-responsive:info` — print help and usage reference
- `/figma-responsive:implement --desktop <url> --mobile <url> [--tablet <url>] [--name Name]` — generate responsive component

**Skill:** `figma-responsive` — auto-activates during implementation; carries the generation logic, step-by-step process, and RSC/cva rules.

**Key reference files** (in `skills/figma-responsive/references/`):
- `vanbrunt-conventions.md` — project-specific Tailwind tokens, cva patterns, RSC rules. **Replace this file** when using the plugin in a project other than vanbrunt.
- `responsive-patterns.md` — breakpoint switching patterns, mobile-first structure, layout decision tree.

**Generated output per component:**
```
src/design-system/<ComponentName>/
  <ComponentName>.tsx           ← responsive RSC (Tailwind + cva)
  <ComponentName>.client.tsx    ← 'use client' leaf (only if interactive)
  use<ComponentName>.ts         ← shared hook (only if stateful)
  <ComponentName>.utils.ts      ← pure helpers (only if needed)
  <ComponentName>.test.tsx      ← stub
  <ComponentName>.stories.tsx   ← Storybook with viewport stories
  index.ts                      ← re-exports
```

### arch-sync

Solves the multi-repo context problem: Claude only knows the repo it's currently in. `arch-sync` reads `CLAUDE.md`, `INTEGRATION.md`, and `README.md` from sibling repos, caches them in `.claude/system-context/<repo-name>/`, and generates a unified `ARCHITECTURE.md` from the current repo's perspective.

**Commands:**
- `/arch-sync:info` — print help reference
- `/arch-sync:describe [--claude-only] [--integration-only] [--force] [--context <desc>]` — analyze current repo and generate `CLAUDE.md` and/or `INTEGRATION.md`
- `/arch-sync:init [--path <dir>] [--yes]` — detect sibling repos and create `arch-sync.json`
- `/arch-sync:sync [--force] [--repo <name>] [--no-copy]` — pull docs from all configured repos and regenerate `ARCHITECTURE.md`

**Skill:** `system-architecture` — generates `ARCHITECTURE.md` by synthesizing cached sibling repo docs. Used internally by `/arch-sync:sync`.

**Key rule:** `ARCHITECTURE.md` is generated — never edit it directly. Update the source `CLAUDE.md` or `INTEGRATION.md` in the relevant repo, then re-run `/arch-sync:sync`.

**Config file** (`arch-sync.json` at project root):
```json
{
  "currentRepo": { "name": "my-repo", "description": "..." },
  "repos": [
    { "name": "mcp", "path": "../mcp" },
    { "name": "ui", "path": "../ui", "files": ["CLAUDE.md", "README.md"] }
  ]
}
```
`files` defaults to `["CLAUDE.md", "INTEGRATION.md", "README.md"]` when omitted.
