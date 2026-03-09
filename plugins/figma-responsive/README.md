# figma-responsive

**Generate responsive React/Next.js components from multiple Figma frames.**

When implementing Figma designs for desktop and mobile, the second viewport always needs manual adjustment to stay consistent with the first. This plugin eliminates that by accepting both frames simultaneously and generating a single unified component with proper Tailwind breakpoints, `cva` variants, shared hooks, and the full design-system file set.

## How It Works

1. You copy two (or three) Figma frame links — desktop, mobile, and optionally tablet
2. The plugin fetches each design via your local Figma MCP server
3. It inspects your project's `tailwind.config.js` and existing component patterns
4. It generates a complete responsive component following your project's conventions

## Prerequisites

- **Figma MCP server** running locally — see [Figma MCP Server setup guide](https://developers.figma.com/docs/figma-mcp-server/) for installation instructions. Run `claude mcp list` to verify it's active.
- Next.js project with Tailwind CSS and `cva` installed
- Figma links from the Figma desktop or web app ("Copy link to selection")

## Setup (for projects other than vanbrunt)

The plugin ships with `skills/figma-responsive/references/vanbrunt-conventions.md` pre-filled with vanbrunt/podium.com tokens (`stinkbug`, `eggshell`, `rust`, custom fonts). If you're using a different project:

1. Open `skills/figma-responsive/references/vanbrunt-conventions.md`
2. Replace the color tokens, font families, breakpoints, and cva patterns with your own project's values
3. The plugin reads this file at generation time to produce correctly-tokenized output

## Installation

```bash
claude plugin install figma-responsive@local-plugins
```

## Quick Start

```
/figma-responsive:implement \
  --desktop https://figma.com/design/abc123/Site?node-id=10:20 \
  --mobile  https://figma.com/design/abc123/Site?node-id=10:30 \
  --name HeroBanner
```

## Commands

| Command | Description |
|---|---|
| `/figma-responsive:implement` | Generate responsive component from Figma frames |
| `/figma-responsive:info` | Show all commands and usage |

## Generated Output

```
src/design-system/<ComponentName>/
  <ComponentName>.tsx            ← responsive RSC (Tailwind + cva breakpoints)
  <ComponentName>.client.tsx     ← 'use client' leaf (only if interactive)
  use<ComponentName>.ts          ← shared hook (only if stateful)
  <ComponentName>.utils.ts       ← pure helpers (only if needed)
  <ComponentName>.test.tsx       ← test stub
  <ComponentName>.stories.tsx    ← Storybook with viewport stories
  index.ts                       ← re-exports
```

## Options

| Flag | Required | Description |
|---|---|---|
| `--desktop <url>` | ✅ | Figma link for desktop frame |
| `--mobile <url>` | ✅ | Figma link for mobile frame |
| `--tablet <url>` | optional | Figma link for tablet frame |
| `--name <Name>` | optional | Component name (PascalCase); inferred from frame if omitted |
