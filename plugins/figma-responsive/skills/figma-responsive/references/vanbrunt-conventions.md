# Project Conventions

> **Project-specific file.** This file contains design tokens, font variables, and component patterns for the `vanbrunt` project (podium.com marketing site). If using this plugin in a different project, replace the contents of this file with your project's own Tailwind tokens, font families, breakpoints, and component patterns. SKILL.md Step 2 reads this file to match generated code to the target project's conventions.

Reference for generating components that follow the vanbrunt (podium.com) Next.js codebase conventions exactly.

## Design Token Reference

### Colors (`tailwind.config.js` → `config/tailwind/theme.colors.js`)

| Token | Default | Shades | Use |
|---|---|---|---|
| `stinkbug` | `#1c1d18` | 50–300 | Near-black brand color; text, borders |
| `eggshell` | `#d9ccbc` | 50–600 | Off-white backgrounds |
| `rust` | `#af4e30` | 50–600 | Primary CTA orange; buttons, accents |
| `dustyblue` | varies | 50–500 | Hover/focus states |

**Never use arbitrary color values** (`text-[#1c1d18]`). Always use token names.

### Font Families

```css
font-display  → var(--font-graphik)   /* Headings, buttons, labels */
font-sans     → var(--font-grenette)  /* Body text */
font-mono     → var(--font-supplymono) /* Code */
```

In Tailwind: `font-display`, `font-sans`, `font-mono`

### Spacing & Sizing

Use the predefined scale only: `p-4`, `gap-6`, `mt-8`, etc. Arbitrary values like `pt-[3.5px]` are a lint error. When Figma shows an exact pixel value, round to the nearest scale step.

### Breakpoints

```js
// Standard Tailwind defaults
sm:  640px
md:  768px   ← tablet breakpoint
lg:  1024px  ← desktop breakpoint
xl:  1280px
2xl: 1536px
```

---

## cva Pattern (class-variance-authority)

Follow the `Button.tsx` canonical pattern exactly:

```tsx
import { cva } from 'class-variance-authority';
import clsx from 'clsx';

const componentDesign = cva(
  // Base classes (always applied):
  'inline-flex items-center font-display transition ease-in-out',
  {
    variants: {
      color: {
        primary: '',
        secondary: '',
        inverted: '',
      },
      size: {
        md: 'py-4 text-button',
        sm: 'py-3 text-button-sm',
      },
    },
    compoundVariants: [
      {
        color: 'primary',
        size: 'md',
        className: clsx(
          'bg-stinkbug-300 text-eggshell-50',
          'hover:bg-dustyblue-100',
        ),
      },
    ],
    defaultVariants: {
      color: 'primary',
      size: 'md',
    },
  },
);
```

**For responsive viewport variants:**
```tsx
const componentLayout = cva('', {
  variants: {
    viewport: {
      mobile: 'block lg:hidden',
      desktop: 'hidden lg:block',
      tablet: 'hidden md:block lg:hidden', // only if tablet frame provided
    },
  },
});
```

---

## Component File Set

Every design-system component must include:

```
src/design-system/<ComponentName>/
  <ComponentName>.tsx        ← named export component
  <ComponentName>.test.tsx   ← co-located unit test (Arrange-Act-Assert)
  <ComponentName>.stories.tsx ← Storybook story (required, tags: ['autodocs'])
  index.ts                   ← re-exports only
```

Optional additions (create only if needed):
```
  <ComponentName>.client.tsx   ← 'use client' interactive leaf
  use<ComponentName>.ts        ← stateful hook ('use client')
  <ComponentName>.utils.ts     ← pure helpers (no client marker)
  <ComponentName>.utils.test.ts ← utils unit test
```

---

## React Server Components Rules

- **Default: RSC** — no `'use client'` directive unless the component needs browser APIs or React state
- **Add `'use client'` only for**:
  - Components using `useState`, `useEffect`, `useReducer`
  - Components using browser APIs (`window`, `document`, `localStorage`)
  - Components using event handlers that cannot be serialized (callbacks with closures)
- **RSC parent + client leaf** is the preferred pattern: RSC handles data/layout, client component handles interaction
- **Never** add `'use client'` to a component just because it renders a client child — only the child needs the directive

---

## TypeScript Conventions

- `"strict": true` — no `any`; use `unknown` + Zod or type guards
- All types exported from `*.types.ts` or derived from `z.infer<typeof schema>` in `*.schema.ts`
- Never define types inline in component files or API files
- Named exports everywhere; default exports only where Next.js requires (`page.tsx`, `layout.tsx`, `normalizer.ts`)
- Import order: 1) Node built-ins, 2) external packages, 3) internal `@/` imports, 4) relative imports
- Path alias: `@/` → `src/`

**Polymorphic components** (when an element type is configurable):
```tsx
import type { PolymorphicProps } from 'src/design-system/utils.types';
// Use PolymorphicProps<E extends ElementType> pattern
```

---

## Images

Always use `next/image` — never `<img>`:

```tsx
import Image from 'next/image';

<Image
  src={src}
  alt={alt}
  width={width}
  height={height}
  className="..."
/>
```

Allowed image hostnames are configured in `next.config.js`. For CMS images from `cms.podium.com`, the hostname is already whitelisted.

---

## Storybook Stories

Every component needs stories with `tags: ['autodocs']` and representative variants:

```tsx
import type { Meta, StoryObj } from '@storybook/react';
import { HeroBanner } from './HeroBanner';

const meta: Meta<typeof HeroBanner> = {
  title: 'Design System/HeroBanner',
  component: HeroBanner,
  tags: ['autodocs'],
  parameters: {
    layout: 'fullscreen', // for full-width sections
  },
};
export default meta;
type Story = StoryObj<typeof HeroBanner>;

export const Default: Story = {
  args: {
    // representative props
  },
};

export const MobileView: Story = {
  args: { /* same */ },
  parameters: {
    viewport: { defaultViewport: 'mobile1' },
  },
};

export const DesktopView: Story = {
  args: { /* same */ },
  parameters: {
    viewport: { defaultViewport: 'desktop' },
  },
};
```

---

## Semantic HTML Requirements

- `<section>` — page sections
- `<article>` — self-contained content
- `<nav>` — navigation
- `<header>` / `<footer>` — within sections
- `<h1>`–`<h6>` — heading hierarchy (do not skip levels)
- `<button>` — interactive button elements (never `<div onClick>`)
- `<a>` — links
- `aria-label` on icon-only buttons
- `aria-hidden="true"` on decorative images
