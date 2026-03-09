---
description: Show all figma-responsive plugin commands, their options, and usage examples
allowed-tools: []
---

# figma-responsive:info

Print a concise help reference for the figma-responsive plugin.

## What to output

Print the following text verbatim (render as markdown):

---

## figma-responsive — Responsive Figma-to-Code Generator

Generates unified responsive React/Next.js components from multiple Figma frames. Accepts desktop + mobile (+ optional tablet) links and produces one component with Tailwind breakpoints, shared hooks, and the full design-system file set.

---

### Commands

#### `/figma-responsive:implement` — Generate a responsive component

Fetches design data from each Figma frame via MCP, inspects your project's conventions, and generates the complete component file set.

```
/figma-responsive:implement --desktop <url>[,<url>...] --mobile <url>[,<url>...] [--tablet <url>[,<url>...]] [--name ComponentName]
```

**Required flags:**
- `--desktop <url>[,<url>...]` — One or more Figma links for the desktop viewport
- `--mobile <url>[,<url>...]` — One or more Figma links for the mobile viewport

**Optional flags:**
- `--tablet <url>[,<url>...]` — One or more Figma links for the tablet viewport (adds mid breakpoint)
- `--name <ComponentName>` — PascalCase component name (inferred from Figma frame name if omitted)

Pass **multiple comma-separated URLs** per breakpoint to represent different visual states (open/closed, expanded/collapsed, active/default). The agent will infer state names from Figma frame names and generate `cva` variants + a stateful hook automatically.

**Examples:**
```
# Desktop + mobile only
/figma-responsive:implement \
  --desktop https://figma.com/design/abc123/Site?node-id=10:20 \
  --mobile  https://figma.com/design/abc123/Site?node-id=10:30

# With tablet and explicit name
/figma-responsive:implement \
  --desktop https://figma.com/design/abc123/Site?node-id=10:20 \
  --mobile  https://figma.com/design/abc123/Site?node-id=10:30 \
  --tablet  https://figma.com/design/abc123/Site?node-id=10:25 \
  --name HeroBanner

# Modal with open/closed states per breakpoint
/figma-responsive:implement \
  --desktop "https://figma.com/design/abc123/Site?node-id=10:20,https://figma.com/design/abc123/Site?node-id=10:21" \
  --mobile  "https://figma.com/design/abc123/Site?node-id=10:30,https://figma.com/design/abc123/Site?node-id=10:31" \
  --name SubscribeModal
```

**Produces:**
```
src/design-system/<ComponentName>/
  <ComponentName>.tsx            ← RSC orchestrator: CSS-switches between view components
  <ComponentName>.mobile.tsx     ← mobile-only layout (no responsive prefixes)
  <ComponentName>.tablet.tsx     ← tablet-only layout (only when --tablet provided)
  <ComponentName>.desktop.tsx    ← desktop-only layout (no responsive prefixes)
  <ComponentName>.client.tsx     ← 'use client' shared interactive leaf (only if stateful)
  use<ComponentName>.ts          ← shared hook: state, handlers, requests (shared by all views)
  <ComponentName>.utils.ts       ← pure helpers (only if needed)
  <ComponentName>.test.tsx       ← test stub
  <ComponentName>.stories.tsx    ← Storybook with per-viewport + state stories
  index.ts                       ← named re-exports
```

Each view file (`mobile`, `tablet`, `desktop`) is written for its viewport only — no `lg:`, `md:` overrides inside them. The orchestrator handles breakpoint switching via CSS visibility. All functional logic (form handlers, triggers, API requests) lives in the shared hook and is consumed by all views.

---

#### `/figma-responsive:info` — Show this help

```
/figma-responsive:info
```

---

### How It Works

1. **Fetches** each Figma frame via your local Figma MCP server; detects states when multiple URLs per breakpoint are provided
2. **Inspects** your project — reads `tailwind.config.js` (including breakpoints), existing design-system components, and `package.json`
3. **Plans** architecture — separate view file per breakpoint, shared hook for all functional logic
4. **Maps** Figma design values to your project's Tailwind tokens (never arbitrary values)
5. **Generates** isolated view components (each written for one viewport only), an orchestrator that CSS-switches between them, and a shared hook consumed by all views

---

### Prerequisites

- Figma MCP server must be running (`claude mcp list` to verify)
- Figma links must be "copy link to selection" URLs (right-click a frame in Figma → Copy link)
- Project should have `tailwind.config.js` and `cva` installed for best results

---

### Breakpoint Reference

Breakpoints are resolved at generation time by reading the `screens` key in the project's `tailwind.config.js`. If no custom screens are defined, the Tailwind defaults are used as fallback:

| Breakpoint | Prefix | Default Width |
|---|---|---|
| Mobile (default) | *(none)* | < 640px |
| Small | `sm:` | ≥ 640px |
| Tablet | `md:` | ≥ 768px |
| Desktop | `lg:` | ≥ 1024px |
| Wide | `xl:` | ≥ 1280px |

When a project defines custom breakpoints, the generated code will use those names and values instead.

---

Run `/figma-responsive:info` any time you need a reminder.
