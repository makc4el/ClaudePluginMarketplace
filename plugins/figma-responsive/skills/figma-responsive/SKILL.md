---
name: figma-responsive
description: This skill should be used when generating responsive React/Next.js components from multiple Figma frames, when the user says "implement figma design for desktop and mobile", "generate responsive component from figma", "convert figma to responsive code", or when /figma-responsive:implement is invoked with desktop and mobile Figma links.
version: 0.1.0
---

# Figma Responsive Implementation

Generates a unified responsive React/Next.js component from multiple Figma frames (desktop + mobile, optionally tablet). Produces separate layout markup per breakpoint using Tailwind CSS with `cva`, shared JS/TS logic extracted to hooks and utils, and the full design-system file set.

## Overview

When implementing Figma designs for multiple screen sizes, each frame contains layout, spacing, and visual rules specific to that viewport. This skill synthesizes all provided frames into one coherent component — avoiding the drift that occurs when desktop and mobile are implemented separately.

**Output per component:**
```
src/design-system/<ComponentName>/
  <ComponentName>.tsx           ← RSC orchestrator: CSS-switches between view components
  <ComponentName>.mobile.tsx    ← mobile-only layout (no responsive prefixes inside)
  <ComponentName>.tablet.tsx    ← tablet-only layout (only if tablet frame provided)
  <ComponentName>.desktop.tsx   ← desktop-only layout (no responsive prefixes inside)
  <ComponentName>.client.tsx    ← shared 'use client' interactive leaf (only if stateful)
  use<ComponentName>.ts         ← shared hook: state, handlers, requests (all views consume this)
  <ComponentName>.utils.ts      ← pure transformation helpers (only if needed)
  <ComponentName>.test.tsx      ← co-located test stub
  <ComponentName>.stories.tsx   ← Storybook stories with per-viewport + state variants
  index.ts                      ← named re-exports
```

**Core rule:** Each view file is viewport-isolated — written as if it's the only layout, with no `lg:`, `md:`, or other responsive prefixes inside it. The orchestrator (`<ComponentName>.tsx`) is the only file that uses breakpoint CSS classes, purely for visibility switching. All functional logic is centralised in `use<ComponentName>.ts` and shared across all views.

## Step 1: Fetch Figma Designs

Parse each Figma URL to extract `fileKey` and `nodeId`:

- Figma URL format: `https://www.figma.com/design/{fileKey}/Title?node-id={nodeId}`
- The `node-id` parameter uses URL encoding — decode `%3A` → `:` and `%2C` → `,`
- `nodeId` format is typically `123:456`

Call the Figma MCP tool for each frame (one call per URL). Common tool names: `get_figma_data`, `figma_get_node`, or check available MCP tools via the tool list. Pass `fileKey` and `nodeId` as separate parameters.

Collect the returned design data for each frame. Note what each frame contains: layout structure, component instances, text, images, interactive elements (forms, buttons, inputs).

### Multi-frame state detection

When more than one URL is provided for a single breakpoint, the frames represent different **visual states** of the same component. After fetching all frames for that breakpoint:

1. **Name the states** using Figma frame names as the primary signal (e.g. "Modal / Open" → `open`, "Toggle – Active" → `active`). If names are ambiguous, diff the frames visually and name based on what changes (overlay visible, icon rotated, content expanded).
2. **Build a state map** listing each state name, its source frame, and the visual differences from the default state.
3. **Flag the component as stateful** — multi-state components always require `.client.tsx` and a `use<ComponentName>.ts` hook regardless of whether other interactive elements are present.
4. **Plan cva state variants** — each detected state becomes a `cva` variant key. The visual diff per state drives which Tailwind classes are applied in each variant.

## Step 2: Inspect Project Conventions

Before generating code, read the target project to match its exact patterns:

1. Read `tailwind.config.js` — identify custom color tokens, font families, spacing scale, breakpoints
2. Read `src/design-system/Button/Button.tsx` (or similar) — understand the `cva` pattern in use
3. Read `package.json` — confirm `cva`, `clsx`, `next`, TypeScript versions
4. Check one existing design-system component (`src/design-system/`) to verify import patterns, export style, and prop types
5. Read the `screens` key from `tailwind.config.js` and record all defined breakpoints. If the project has no custom `screens`, fall back to Tailwind defaults: `sm: 640px`, `md: 768px`, `lg: 1024px`, `xl: 1280px`. Use the actual project breakpoint values (not hardcoded defaults) in all generated code — including comments, Storybook viewport configs, and the summary report.

Use these conventions exactly. Do not invent patterns not present in the codebase.

## Step 3: Plan Component Architecture

Determine the structure before writing code:

**View files per breakpoint:**
- Always create a separate layout file for each provided viewport: `.mobile.tsx`, `.tablet.tsx` (if tablet frame provided), `.desktop.tsx`
- Each view file is viewport-isolated — no responsive prefixes (`lg:`, `md:`, etc.) inside it
- `<ComponentName>.tsx` is the RSC orchestrator — contains only CSS-visibility switching, no layout logic

**Shared functional logic:**
- If the design has forms, inputs, toggles, API requests, or multiple state frames → create `use<ComponentName>.ts` (shared hook) + `<ComponentName>.client.tsx` (shared interactive leaf imported by any view that needs it)
- If only pure data transforms are needed → `<ComponentName>.utils.ts`; importable in any view
- If no interactivity → all view files are pure RSC; skip hook and client leaf entirely

**Responsive strategy:**
- Generate a separate layout file per breakpoint (`mobile`, `tablet`, `desktop`) — each written for its viewport only with no responsive prefixes inside
- The orchestrator wraps each view in a CSS-visibility div using the project's breakpoint prefixes (from Step 2):
  ```tsx
  // two breakpoints (no tablet)
  <div className="block lg:hidden"><ComponentNameMobile {...props} /></div>
  <div className="hidden lg:block"><ComponentNameDesktop {...props} /></div>

  // three breakpoints (tablet provided)
  <div className="block md:hidden"><ComponentNameMobile {...props} /></div>
  <div className="hidden md:block lg:hidden"><ComponentNameTablet {...props} /></div>
  <div className="hidden lg:block"><ComponentNameDesktop {...props} /></div>
  ```
- No responsive prefixes (`lg:`, `md:`, etc.) appear inside any view file — only in the orchestrator

## Step 4: Map Design Tokens

Map Figma design values to the project's Tailwind tokens. Do not use arbitrary values like `text-[#1c1d18]` — use token names.

Consult `references/vanbrunt-conventions.md` for the full token map (colors, fonts, spacing). **Note:** this file is project-specific — it contains vanbrunt tokens by default. Replace it with your project's own tokens if working in a different codebase. Key mappings for vanbrunt:
- Near-black `#1c1d18` → `stinkbug-300` or `text-stinkbug-300`
- Off-white `#d9ccbc` → `eggshell` / `bg-eggshell`
- CTA orange `#af4e30` → `rust` / `bg-rust`
- Headings → `font-display`
- Body text → `font-sans`

When a Figma color has no project token equivalent, check if it's a close match to an existing token shade before using an arbitrary value.

## Step 5: Generate Files

### `<ComponentName>.tsx` (RSC orchestrator)

The orchestrator has no layout logic — only breakpoint visibility switching. Use the project's actual breakpoint prefixes from Step 2.

```tsx
// No 'use client' — this is an RSC
import { <ComponentName>Mobile } from './<ComponentName>.mobile';
import { <ComponentName>Desktop } from './<ComponentName>.desktop';
// import { <ComponentName>Tablet } from './<ComponentName>.tablet'; // only if tablet frame provided

interface <ComponentName>Props {
  // Shared prop types passed to all views
}

export function <ComponentName>(props: <ComponentName>Props) {
  return (
    <>
      <div className="block lg:hidden">
        <<ComponentName>Mobile {...props} />
      </div>
      {/* tablet — only if tablet frame was provided:
      <div className="hidden md:block lg:hidden">
        <<ComponentName>Tablet {...props} />
      </div> */}
      <div className="hidden lg:block">
        <<ComponentName>Desktop {...props} />
      </div>
    </>
  );
}
```

### `<ComponentName>.mobile.tsx` / `.tablet.tsx` / `.desktop.tsx` (view files)

Each view is written for its viewport only. No responsive prefixes inside — just static Tailwind classes matching that viewport's Figma frame.

```tsx
// No 'use client' — RSC by default
// No lg:, md:, sm: prefixes — this file is for [mobile|tablet|desktop] only

interface <ComponentName>MobileProps {
  // Same shape as the orchestrator props
}

export function <ComponentName>Mobile({ ...props }: <ComponentName>MobileProps) {
  return (
    <section className="px-4 py-8">
      {/* Mobile-specific markup, no responsive overrides */}
    </section>
  );
}
```

If the view needs interactivity, import the shared client leaf:
```tsx
import { <ComponentName>Form } from './<ComponentName>.client';
```

### `<ComponentName>.client.tsx` (shared interactive leaf, only if stateful)

One client component shared by all views. Imports the shared hook.

```tsx
'use client';

import { use<ComponentName> } from './use<ComponentName>';

export function <ComponentName>Form() {
  const { handleSubmit, fields, errors } = use<ComponentName>();
  return <form onSubmit={handleSubmit}>...</form>;
}
```

### `use<ComponentName>.ts` (shared hook, only if stateful)

All state, event handlers, and API requests live here. Consumed by any view via the shared client leaf.

```ts
'use client';

export function use<ComponentName>() {
  // useState, useCallback, form handling, API calls
  // This hook is the single source of truth for all views
  return { handleSubmit, fields, errors };
}
```

### `<ComponentName>.test.tsx` (stub)

```tsx
import { render, screen } from '@testing-library/react';
import { <ComponentName> } from './<ComponentName>';

describe('<ComponentName>', () => {
  it('renders correctly', () => {
    render(<<ComponentName> />);
    // TODO: add assertions
  });
});
```

### `<ComponentName>.stories.tsx` (Storybook stub)

```tsx
import type { Meta, StoryObj } from '@storybook/react';
import { <ComponentName> } from './<ComponentName>';

const meta: Meta<typeof <ComponentName>> = {
  title: 'Design System/<ComponentName>',
  component: <ComponentName>,
  tags: ['autodocs'],
};
export default meta;
type Story = StoryObj<typeof <ComponentName>>;

export const Default: Story = {};
export const Mobile: Story = {
  parameters: { viewport: { defaultViewport: 'mobile1' } },
};
export const Desktop: Story = {
  parameters: { viewport: { defaultViewport: 'desktop' } },
};
```

### `index.ts`

```ts
export { <ComponentName> } from './<ComponentName>';
// Export client component and hook if created
```

## Step 6: Report Output

After writing all files, print a summary:

```
✅ Generated: src/design-system/HeroBanner/

Files created:
  HeroBanner.tsx           ← RSC orchestrator (CSS visibility switching)
  HeroBanner.mobile.tsx    ← mobile-only layout
  HeroBanner.tablet.tsx    ← tablet-only layout        [if tablet provided]
  HeroBanner.desktop.tsx   ← desktop-only layout
  HeroBanner.client.tsx    ← shared interactive leaf   [if stateful]
  useHeroBanner.ts         ← shared hook (all views)   [if stateful]
  HeroBanner.utils.ts      ← pure helpers              [if needed]
  HeroBanner.test.tsx      ← test stub
  HeroBanner.stories.tsx   ← per-viewport + state stories
  index.ts                 ← re-exports

Breakpoints (from tailwind.config.js): mobile | [tablet (md:) |] desktop (lg:)
Frames used: desktop ✅ (N)  mobile ✅ (N)  [tablet ✅ (N)]
[States detected: open | closed   → cva variants in each view, shared hook]
```

## Additional Resources

- **`references/vanbrunt-conventions.md`** — Tailwind design tokens, cva patterns, RSC rules, TypeScript conventions specific to the vanbrunt project
- **`references/responsive-patterns.md`** — Breakpoint switching patterns, mobile-first structure, tablet handling, shared logic decision tree
