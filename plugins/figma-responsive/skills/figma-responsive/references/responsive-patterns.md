# Responsive Component Patterns

Reference for structuring responsive React/Next.js components with multiple Figma frame sources.

## Breakpoint Strategy

### Mobile-First Approach

All styles default to mobile. Larger viewports override using Tailwind prefixes.

**Breakpoints are read from the project's `tailwind.config.js` (`screens` key) before generating any code.** Use the actual prefix names and pixel values from the project. If no custom `screens` are defined, fall back to Tailwind defaults:

```
default     → mobile  (< 640px)
sm:         → (≥ 640px)
md:         → tablet  (≥ 768px)  — only when tablet frame is provided
lg:         → desktop (≥ 1024px)
xl:         → (≥ 1280px)
```

**Tailwind responsive prefix usage:**
```tsx
// Text sizes
<h1 className="text-h2 lg:text-h1">

// Spacing
<section className="px-4 md:px-8 lg:px-16 py-12 lg:py-24">

// Grid
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">

// Flex direction
<div className="flex flex-col lg:flex-row items-start gap-8">
```

---

## Split-View Architecture

Each breakpoint gets its own isolated layout file. The orchestrator is the only place that uses breakpoint CSS classes — purely for visibility switching.

### File structure

```
<ComponentName>/
  <ComponentName>.tsx           ← RSC orchestrator (visibility only)
  <ComponentName>.mobile.tsx    ← mobile layout (no responsive prefixes)
  <ComponentName>.tablet.tsx    ← tablet layout (no responsive prefixes) — optional
  <ComponentName>.desktop.tsx   ← desktop layout (no responsive prefixes)
  <ComponentName>.client.tsx    ← shared interactive leaf — optional
  use<ComponentName>.ts         ← shared hook (state, handlers, requests) — optional
```

### Orchestrator pattern (two breakpoints)

```tsx
// <ComponentName>.tsx — RSC, no layout logic
import { <ComponentName>Mobile } from './<ComponentName>.mobile';
import { <ComponentName>Desktop } from './<ComponentName>.desktop';

export function <ComponentName>(props: <ComponentName>Props) {
  return (
    <>
      <div className="block lg:hidden">
        <<ComponentName>Mobile {...props} />
      </div>
      <div className="hidden lg:block">
        <<ComponentName>Desktop {...props} />
      </div>
    </>
  );
}
```

### Orchestrator pattern (three breakpoints)

```tsx
// <ComponentName>.tsx — RSC, no layout logic
import { <ComponentName>Mobile } from './<ComponentName>.mobile';
import { <ComponentName>Tablet } from './<ComponentName>.tablet';
import { <ComponentName>Desktop } from './<ComponentName>.desktop';

export function <ComponentName>(props: <ComponentName>Props) {
  return (
    <>
      <div className="block md:hidden">
        <<ComponentName>Mobile {...props} />
      </div>
      <div className="hidden md:block lg:hidden">
        <<ComponentName>Tablet {...props} />
      </div>
      <div className="hidden lg:block">
        <<ComponentName>Desktop {...props} />
      </div>
    </>
  );
}
```

### View file rules

- **No responsive prefixes** (`lg:`, `md:`, `sm:`, etc.) inside any view file
- Write each view as if it's the only layout that will ever render
- Import the shared client leaf if the view needs interactivity
- Props interface is identical across all views (same shape as orchestrator props)

```tsx
// <ComponentName>.mobile.tsx — mobile only, no responsive prefixes
export function <ComponentName>Mobile({ title, image, cta }: <ComponentName>Props) {
  return (
    <section className="px-4 py-8">
      <Image src={image} alt="..." width={375} height={250} />
      <h1 className="font-display text-h2 text-stinkbug-300">{title}</h1>
      <CtaButton href={cta.href}>{cta.label}</CtaButton>
    </section>
  );
}

// <ComponentName>.desktop.tsx — desktop only, no responsive prefixes
export function <ComponentName>Desktop({ title, image, cta }: <ComponentName>Props) {
  return (
    <section className="px-16 py-24">
      <div className="grid grid-cols-2 gap-12">
        <div>
          <h1 className="font-display text-h1 text-stinkbug-300">{title}</h1>
          <CtaButton href={cta.href}>{cta.label}</CtaButton>
        </div>
        <Image src={image} alt="..." width={600} height={500} />
      </div>
    </section>
  );
}
```

### Shared logic pattern

All functional logic (state, event handlers, API requests) lives in `use<ComponentName>.ts` and is consumed by all views through the shared `.client.tsx` leaf. Never duplicate logic across view files.

```tsx
// <ComponentName>.client.tsx — one client component, shared by all views
'use client';
import { use<ComponentName> } from './use<ComponentName>';

export function <ComponentName>Form() {
  const { handleSubmit, fields, errors } = use<ComponentName>();
  return <form onSubmit={handleSubmit}>...</form>;
}

// Any view file that needs the form just imports it:
// import { <ComponentName>Form } from './<ComponentName>.client';
```

---

## Shared Logic Decision Tree

```
Is there user interaction? (form, toggle, modal, dropdown)
├── YES: Does it require React state?
│   ├── YES: Create use<ComponentName>.ts hook + <ComponentName>.client.tsx
│   └── NO (pure handlers): Create <ComponentName>.utils.ts
└── NO: No .client.tsx or hook needed — pure RSC
```

**Hook template (`use<ComponentName>.ts`):**
```ts
'use client';

import { useState, useCallback } from 'react';

interface Use<ComponentName>Return {
  // Typed return shape
}

export function use<ComponentName>(): Use<ComponentName>Return {
  const [isLoading, setIsLoading] = useState(false);

  const handleSubmit = useCallback(async (data: FormData) => {
    setIsLoading(true);
    try {
      // form submission logic
    } finally {
      setIsLoading(false);
    }
  }, []);

  return { isLoading, handleSubmit };
}
```

**Utils template (`<ComponentName>.utils.ts`):**
```ts
// No 'use client' — pure functions, safe to import in RSC

export function formatPhoneNumber(raw: string): string {
  // ...
}

export function truncateText(text: string, maxLength: number): string {
  // ...
}
```

---

## RSC + Client Component Composition

```tsx
// HeroBanner.tsx (RSC — no 'use client')
import { HeroBannerForm } from './HeroBanner.client';

export function HeroBanner({ title, formAction }: HeroBannerProps) {
  return (
    <section className="...">
      <h1 className="font-display text-h1 text-stinkbug-300">{title}</h1>
      {/* RSC passes serializable props to client component */}
      <HeroBannerForm action={formAction} />
    </section>
  );
}

// HeroBanner.client.tsx
'use client';

import { useHeroBanner } from './useHeroBanner';

interface HeroBannerFormProps {
  action: string;
}

export function HeroBannerForm({ action }: HeroBannerFormProps) {
  const { handleSubmit, fields, isLoading } = useHeroBanner({ action });

  return (
    <form onSubmit={handleSubmit} className="...">
      {/* controlled inputs */}
    </form>
  );
}
```

**Key constraint:** Props passed from RSC to client component must be serializable (no functions, no class instances, no Dates unless stringified).

---

## Handling Figma Auto Layout

Map Figma Auto Layout properties to CSS Flexbox/Grid:

| Figma | CSS/Tailwind |
|---|---|
| Horizontal auto layout | `flex flex-row` |
| Vertical auto layout | `flex flex-col` |
| Gap: 16px | `gap-4` |
| Padding: 16px 24px | `py-4 px-6` |
| Fill container | `w-full` |
| Hug contents | (default, no `w-` class needed) |
| Fixed: 400px | `w-[400px]` → prefer `max-w-lg` or similar token |
| Alignment: center | `items-center justify-center` |
| Alignment: space-between | `justify-between` |

---

## Image Handling

Use `next/image` with the correct aspect ratio from Figma:

```tsx
<Image
  src={src}                           // string prop
  alt={alt}                           // always required
  width={designWidth}                 // from Figma frame dimensions
  height={designHeight}               // from Figma frame dimensions
  className="object-cover w-full"     // Tailwind classes
  priority={isAboveTheFold}           // boolean prop for LCP images
/>
```

For background-style images, use `fill` prop with a positioned container:

```tsx
<div className="relative w-full aspect-[16/9]">
  <Image src={src} alt="" fill className="object-cover" />
</div>
```
