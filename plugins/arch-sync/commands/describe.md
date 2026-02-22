---
description: Analyzes the current repo and generates CLAUDE.md (internal dev guide) and INTEGRATION.md (external integration reference) if they are missing or incomplete
argument-hint: "[--claude-only] [--integration-only] [--force] [--context <description>]"
allowed-tools: Read, Write, Bash, Glob
---

# arch-sync: describe

Analyze the current repository and generate `CLAUDE.md` and/or `INTEGRATION.md` based on the actual code, structure, and API surface.

Run this in any repo that is part of your multi-repo system but is missing its documentation — so it can be included in `/arch-sync:sync` and give Claude full system context.

## What this command produces

- **`CLAUDE.md`** — Internal developer guide: what this repo does, directory structure, key patterns, environment variables, common commands, architecture notes. Claude reads this when working in this repo.
- **`INTEGRATION.md`** — External integration reference: how other repos/systems connect to this one. Covers endpoints, data schemas, auth patterns, SDK examples. Claude reads this when working in sibling repos.

## Step-by-step instructions

### Step 1: Check what already exists

Read the current directory to check which files exist:
- Does `CLAUDE.md` exist? If yes and `--force` is not set, skip generating it (tell the user).
- Does `INTEGRATION.md` exist? Same rule.

If `--claude-only`, only generate `CLAUDE.md`. If `--integration-only`, only generate `INTEGRATION.md`.

If both already exist and `--force` is not set, tell the user: "Both CLAUDE.md and INTEGRATION.md already exist. Use --force to regenerate."

### Step 2: Discover the repo

Gather the following about the current repo:

**Identity:**
- Repo/directory name (from `pwd`)
- Language and framework (detect from: `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `pom.xml`, `Gemfile`, etc.)
- Runtime environment (Node.js, Python, Go, etc.)
- Deployment target if discernible (Railway, Vercel, AWS Lambda, LangGraph Platform, etc.)

**Structure:**
- List top-level directories (use Bash `ls -1`)
- Identify the main entry point(s): `main.py`, `index.ts`, `server.ts`, `app.py`, `main.go`, etc.
- Identify config files: `.env.template` / `.env.example`, `langgraph.json`, `docker-compose.yml`, `railway.json`, etc.

**Interfaces (for INTEGRATION.md):**
- **HTTP/REST**: Read main server file(s) for route definitions (`app.get`, `router.post`, `@app.route`, etc.)
- **SSE endpoints**: Look for `/sse`, `EventSource`, `text/event-stream` references
- **MCP tools**: Look for tool registration patterns (`server.tool(`, `@mcp.tool`, etc.)
- **LangGraph graphs**: Look for `StateGraph`, graph compilation, exported graph objects
- **CLI commands**: Look for argparse, click, typer definitions
- **Environment variables**: Read `.env.template` / `env.template` / `.env.example` for required variables

**Dependencies:**
- Read `package.json` / `pyproject.toml` / `requirements.txt` for main dependencies
- Identify which sibling repos this repo calls (look for `MCP_SERVER_URL`, `LANGGRAPH_API_URL`, `SUPABASE_URL` references)

If `--context <description>` was provided, treat it as additional context about the repo's role (e.g., "This is the MCP server that wraps Salesforce APIs").

### Step 3: Read key source files

Read the main entry point and 2-4 other key files to understand:
- What the code does (domain, business logic)
- What it exposes (routes, tools, functions, CLI commands)
- What it calls out to (external services, sibling repos)
- Auth patterns used (OAuth, API keys, vault lookup, etc.)
- Data shapes (TypeScript interfaces, Python TypedDicts, Pydantic models, zod schemas)

Do not read every file — focus on the files that define the public interface and core behavior.

### Step 4: Generate CLAUDE.md

Write `CLAUDE.md` using this structure. Adapt and omit sections that don't apply.

```markdown
# CLAUDE.md — <Repo Name>

This file provides guidance to Claude Code when working with this repository.

---

## Project Overview

<2-3 sentences: what this repo does, its role in the wider system, what it's deployed as>

**Deployed on:** <target>
**Main entry point:** <file>

---

## Directory Structure

```
<language-appropriate tree of key directories and files>
```

---

## Key Patterns & Conventions

<3-5 patterns specific to this codebase that a developer must know. Examples:>
- Auth pattern (e.g., "Never pass credentials directly — always use userId for vault lookup")
- Error handling approach
- Naming conventions
- How to add a new feature (e.g., "To add a new MCP tool: create handler in src/tools/, export, register in server.ts")
- State management / data flow patterns

---

## Environment Variables

### Required

| Variable | Purpose |
|---|---|
| <VAR_NAME> | <what it does> |

### Optional

| Variable | Default | Purpose |
|---|---|---|
| <VAR_NAME> | <default> | <what it does> |

---

## Common Commands

```bash
# Install dependencies
<install command>

# Run locally
<dev command>

# Build / test
<build or test command>
```

---

## Architecture Notes

<Any gotchas, known issues, dead code, or important decisions worth flagging>
```

### Step 5: Generate INTEGRATION.md

Write `INTEGRATION.md` using this structure. Only include sections that apply — a CLI-only tool has no API section; a tool with no auth has no auth section.

```markdown
# INTEGRATION.md — <Repo Name>

How to integrate with <Repo Name> from other systems or repos.

---

## Deployment Constants

| | Value |
|---|---|
| **Public URL** | <if known, from env.template or config> |
| **Protocol** | <REST / MCP/SSE / LangGraph Platform / CLI / etc.> |

---

## Authentication

<How callers authenticate to this repo. If none, say "No authentication required.">

Example: "All requests require `userId` parameter — used to fetch OAuth tokens from Supabase Vault."

---

## Interfaces

### <Interface type — e.g., HTTP API / MCP Tools / LangGraph Graph / CLI>

<For each endpoint or tool:>

**`<METHOD> <path>`** or **`<tool_name>`**
- **Purpose**: <what it does>
- **Input**: <parameters or request body shape>
- **Output**: <response shape>
- **Auth**: <auth requirement>

---

## Data Schemas

<Key request/response/event shapes that callers need to know about. Include TypeScript interface or Python TypedDict if discoverable from source.>

---

## Integration Examples

<One curl, SDK snippet, or code example showing how to call the most common operation>

---

## Environment Variables Required by Callers

<Variables that sibling repos must set to connect to this component>

| Variable | Example Value | Purpose |
|---|---|---|
| <VAR_NAME> | <example> | <what callers use it for> |
```

### Step 6: Report what was generated

After writing the files, print:

```
✅ Generated: CLAUDE.md
✅ Generated: INTEGRATION.md

These files are ready to commit.

Next steps:
  1. Review and edit both files — fill in any [UNKNOWN] placeholders
  2. Add this repo to your main system's arch-sync.json
  3. Run /arch-sync:sync in your main repo to pull in these docs and regenerate ARCHITECTURE.md
```

If any sections could not be completed due to missing information, mark them with `<!-- TODO: fill in -->` and list them in the report.

## Argument handling

- `--claude-only`: Only generate `CLAUDE.md`
- `--integration-only`: Only generate `INTEGRATION.md`
- `--force`: Overwrite files even if they already exist
- `--context <description>`: Additional context about this repo's role passed as a hint (e.g., `--context "TypeScript MCP server wrapping Salesforce REST API"`)

## Quality guidelines

**For CLAUDE.md:**
- Write for Claude, not for humans — include things a developer wouldn't write down but Claude needs to know (naming patterns, anti-patterns, how to trace a bug)
- Be specific: name actual files, actual function names, actual patterns from the code
- Keep it under 400 lines — it will be loaded into context on every Claude session in this repo

**For INTEGRATION.md:**
- Write for an external consumer — assume they don't know this codebase at all
- Every endpoint or tool must have a concrete example input/output
- The auth section must be actionable — not "uses OAuth" but "pass userId parameter; server fetches token from Supabase Vault using that userId"
- If you cannot determine an exact value (e.g., the public URL is not in the config), write `<!-- TODO: fill in public URL after deployment -->` rather than guessing

## Error handling

- Cannot read source files → generate with best-effort content based on directory structure + package manifests; mark uncertain sections with `<!-- TODO -->`
- Multiple entry points found → document the primary one and mention others
- No recognizable API surface found → generate minimal INTEGRATION.md with a note: "This component exposes no public API — integrate via [shared database / file system / CLI]"
