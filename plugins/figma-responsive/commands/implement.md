---
description: Generate a responsive React/Next.js component from multiple Figma frames. Accepts desktop + mobile (+ optional tablet) Figma links and produces the full design-system file set with Tailwind breakpoints and shared logic.
argument-hint: "--desktop <url>[,<url>...] --mobile <url>[,<url>...] [--tablet <url>[,<url>...]] [--name ComponentName]"
allowed-tools: [Read, Write, Bash, Glob, get_figma_data, figma_get_node]
---

# figma-responsive:implement

Generate a unified, responsive React/Next.js component from multiple Figma frames using the Figma MCP server.

## Argument Parsing

Parse the following flags from the user's arguments:

- `--desktop <url>[,<url>...]` — One or more Figma links for the desktop viewport (required). Comma-separated for multiple states.
- `--mobile <url>[,<url>...]` — One or more Figma links for the mobile viewport (required). Comma-separated for multiple states.
- `--tablet <url>[,<url>...]` — One or more Figma links for the tablet viewport (optional). Comma-separated for multiple states.
- `--name <ComponentName>` — PascalCase component name (optional; infer from Figma frame name if not provided)

**Multi-state syntax:** Pass multiple comma-separated URLs to represent different visual states of the component at that breakpoint (e.g. open/closed, expanded/collapsed, active/default). The agent will compare the frames and infer state names from the Figma frame names or visual differences.

If `--desktop` or `--mobile` is missing, stop and tell the user:
> "Both `--desktop` and `--mobile` links are required. Example:\n> `/figma-responsive:implement --desktop <url> --mobile <url> --name HeroBanner`"

## Step 1: Fetch Each Figma Frame

For each provided URL (across all breakpoints), extract `fileKey` and `nodeId`:

1. Figma URL format: `https://www.figma.com/design/{fileKey}/Title?node-id={nodeId}`
2. The `node-id` query parameter is URL-encoded. Decode: `%3A` → `:` and `%2C` → `,`
3. Node ID format is typically `123:456`

Call the Figma MCP tool for each frame separately. Try these tool names in order until one works:
- `get_figma_data` with params `{ fileKey, nodeId }`
- `figma_get_node` with params `{ file_key, node_id }`
- Check the available MCP tools list if neither works

If the `--name` flag was not provided, use the Figma frame name from the first desktop frame response as the component name (converted to PascalCase). Strip special characters and spaces.

### State Detection (when multiple URLs per breakpoint)

When more than one URL is provided for a breakpoint, compare the fetched frames to identify component states:

1. **Use Figma frame names as primary signal** — names like "Modal / Open", "Toggle – Active", "Card – Collapsed" directly map to state names. Convert them to camelCase variant names (e.g. `open`, `closed`, `active`, `default`).
2. **Fall back to visual diff** — if frame names are generic, identify which elements appear/disappear or change between frames (e.g. an overlay div, an icon rotation, expanded content area) and name the states accordingly.
3. **Record the state map** — build a table before generating code:

| State name | Breakpoint | Figma frame |
|---|---|---|
| `open` | desktop | "Modal / Open" |
| `closed` | desktop | "Modal / Closed" |

4. **States always require interactivity** — a component with detected states must include `.client.tsx` and `use<ComponentName>.ts`. Do not generate a pure RSC when states are present.

## Step 2: Inspect Project Conventions

Read the target project to match its exact patterns:

1. Use Bash `pwd` to get current working directory
2. Read `tailwind.config.js` — note custom color tokens, font families, breakpoints
3. Glob for an existing design-system component (e.g., `src/design-system/Button/Button.tsx`) and read it — understand the `cva` pattern and export style
4. Read `package.json` — confirm `cva`, `clsx`, `next` are present

Determine component destination:
- If `src/design-system/` exists → place there (shared, reusable component)
- If component is page-specific → place in `src/feature/<feature-name>/`
- Default: `src/design-system/<ComponentName>/`

## Step 3: Plan the Architecture

Before writing any files, decide:

1. **View files per breakpoint** — always create a dedicated layout file for each provided viewport:
   - `<ComponentName>.mobile.tsx` — mobile layout only; no responsive prefixes inside
   - `<ComponentName>.tablet.tsx` — tablet layout only; created only when `--tablet` was provided
   - `<ComponentName>.desktop.tsx` — desktop layout only; no responsive prefixes inside
   - `<ComponentName>.tsx` — RSC orchestrator; wraps each view in a CSS-visibility div using the project's breakpoint prefixes (e.g. `block md:hidden`, `hidden md:block lg:hidden`, `hidden lg:block`)

2. **Shared functional logic** — Does the design contain forms, inputs, triggers, API requests, toggles, or multiple state frames?
   - YES → create `use<ComponentName>.ts` (shared hook, used by all views) + `<ComponentName>.client.tsx` (interactive leaf imported by any view that needs it)
   - NO → all view files are pure RSC; no hook or client file needed

3. **Pure helpers** — Are there data transforms, formatters, or stateless utilities?
   - YES → `<ComponentName>.utils.ts`; import in whichever views need it
   - NO → skip

4. **State variant strategy** (only when states were detected in Step 1):
   - Use `cva` inside each view file for state variants (e.g. `variant: { open: '...', closed: '...' }`)
   - The shared `use<ComponentName>.ts` hook owns the active state (`useState`) and exposes toggle/setter
   - Views receive current state and setter as props from the orchestrator or the shared client leaf

5. **File set to create**: List all files before writing any of them.

## Step 4: Generate the Component

Follow the full generation process from the `figma-responsive` skill. Key rules:

- Map Figma design values to project Tailwind tokens — never use arbitrary values like `text-[#1c1d18]`
- **View files are viewport-isolated** — write each `.mobile.tsx` / `.tablet.tsx` / `.desktop.tsx` as if it's the only layout; no `lg:`, `md:` overrides inside them
- **Orchestrator only uses breakpoint classes** — `<ComponentName>.tsx` contains no layout logic, only visibility switching via project breakpoint prefixes
- All functional logic (state, handlers, requests) lives in `use<ComponentName>.ts` and is shared by all views; never duplicate logic across view files
- Use `cva` for state variants within each view file, matching the project's existing pattern
- Named exports only (`export function HeroBanner`); no default exports except where Next.js requires
- `aria-*` on interactive elements without visible text labels
- `next/image` for all images

## Step 5: Write All Files

Create the output directory:
```bash
mkdir -p <destination>/<ComponentName>
```

Write each file in sequence:
1. `use<ComponentName>.ts` — shared hook (if stateful; write first so views can import it)
2. `<ComponentName>.client.tsx` — shared interactive leaf (if stateful; skip otherwise)
3. `<ComponentName>.utils.ts` — pure helpers (skip if not needed)
4. `<ComponentName>.mobile.tsx` — mobile-only layout
5. `<ComponentName>.tablet.tsx` — tablet-only layout (skip if `--tablet` not provided)
6. `<ComponentName>.desktop.tsx` — desktop-only layout
7. `<ComponentName>.tsx` — RSC orchestrator: CSS-switches between view components
8. `<ComponentName>.test.tsx` — test stub
9. `<ComponentName>.stories.tsx` — Storybook stories with per-viewport + state stories
10. `index.ts` — re-exports

## Step 6: Print Summary

```
✅ Generated: src/design-system/<ComponentName>/

Files created:
  <ComponentName>.tsx            ← RSC orchestrator (CSS visibility switching)
  <ComponentName>.mobile.tsx     ← mobile-only layout
  [<ComponentName>.tablet.tsx    ← tablet-only layout]
  <ComponentName>.desktop.tsx    ← desktop-only layout
  [<ComponentName>.client.tsx    ← shared interactive leaf]
  [use<ComponentName>.ts         ← shared hook (all views)]
  <ComponentName>.test.tsx       ← test stub
  <ComponentName>.stories.tsx    ← per-viewport + state stories
  index.ts                       ← re-exports

Breakpoints (from tailwind.config.js): mobile | [tablet (md:) |] desktop (lg:)
Frames used: desktop ✅ (N frames)  mobile ✅ (N frames)  [tablet ✅ (N frames)]
[States detected: open | closed   (cva variants, shared hook)]
```

## Error Handling

- **Figma MCP tool not found** → tell the user which MCP tools were tried and ask them to verify the Figma MCP server is running (`claude mcp list`)
- **Invalid Figma URL** → show what was parsed and ask user to paste the full link from the Figma app
- **Component name conflicts** (directory already exists) → ask: "A component named `<X>` already exists at `<path>`. Overwrite, use a different name, or cancel?"
- **Missing project files** (no `tailwind.config.js`) → proceed with sensible defaults but note: "Could not detect project conventions — using standard Next.js/Tailwind patterns"

## Tips

- The Figma link must come from the Figma desktop app or browser. Right-click a frame → "Copy link to selection" to get the exact node URL
- Use `--name` when the Figma frame name is generic (e.g., "Frame 1")
- For page-level layouts, prefer placing in `src/feature/<page-name>/` rather than `src/design-system/`
- Run `/figma-responsive:info` to see all available options
