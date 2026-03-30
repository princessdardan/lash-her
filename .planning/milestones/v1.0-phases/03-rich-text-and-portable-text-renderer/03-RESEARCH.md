# Phase 3: Rich Text and Portable Text Renderer - Research

**Researched:** 2026-03-26
**Domain:** @portabletext/react, Tailwind CSS v4, brand-styled Portable Text rendering
**Confidence:** HIGH

---

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

- **D-01:** Replace `ui/block-renderer.tsx` in-place with the full `@portabletext/react` renderer. Rename file to `portable-text-renderer.tsx`. No `strapi-rich-text-renderer.tsx` needed — the original Strapi renderer was already replaced by a PT stub in Phase 2.
- **D-02:** File location: `frontend/src/components/ui/portable-text-renderer.tsx` — stays in `ui/` consistent with other reusable UI components (sanity-image.tsx, logo.tsx).
- **D-03:** Export name: `PortableTextRenderer` — clear, descriptive, no collision with `layouts/block-renderer.tsx`'s `BlockRenderer` export.
- **D-04:** Use Tailwind Typography `prose` classes as the base, then override specific elements (h2, h3, links, lists) in the `@portabletext/react` component map with brand classes (font-serif, text-brand-red, etc.). Prose handles spacing/defaults, component map handles brand look.
- **D-05:** Links honor the schema's `blank` field: open in new tab when `blank=true` (`target="_blank"` + `rel="noopener noreferrer"`), same tab otherwise.
- **D-06:** One universal renderer — `PortableTextRenderer` renders the same styled PT output everywhere. Parent layout components wrap it in their own container divs with context-specific classes.
- **D-07:** Layout components pass the full Portable Text array: `<PortableTextRenderer content={data.info} />`. The renderer handles the full array internally.

### Claude's Discretion

- Exact Tailwind prose modifier classes and brand overrides in the component map
- Whether to install `@tailwindcss/typography` if not already present, or rely on existing prose classes
- Internal structure of the component map (block, marks, list, listItem handlers)
- How to handle edge cases (empty arrays, malformed blocks)

### Deferred Ideas (OUT OF SCOPE)

None — discussion stayed within phase scope.
</user_constraints>

---

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| RT-01 | Portable Text renderer built using `@portabletext/react` with brand-styled component map matching existing visual output (headings, paragraphs, lists, links, bold, italic) | `@portabletext/react` v6.0.3 API verified; component map structure documented with exact class specifications from 03-UI-SPEC.md |
| RT-02 | Existing `ui/block-renderer.tsx` (rich text renderer) renamed to `strapi-rich-text-renderer.tsx` to eliminate naming collision before migration | Superseded per D-01: file renamed to `portable-text-renderer.tsx` instead; no Strapi renderer collision exists (Phase 2 replaced original); 4 consumer files identified for import updates |
</phase_requirements>

---

## Summary

Phase 3 is a focused component replacement: the plain-text stub in `frontend/src/components/ui/block-renderer.tsx` is renamed to `portable-text-renderer.tsx` and upgraded to a full `@portabletext/react` renderer with brand styling. The schema permits `normal`, `h2`, `h3` block styles; `bullet` and `number` lists; `strong` and `em` decorators; and a `link` annotation with `href` and `blank` fields. All of these must be handled in the component map.

The primary technical finding is that **`@tailwindcss/typography` is not installed**, meaning `prose` classes are currently inert in Tailwind v4. D-04 calls for prose as the spacing baseline — the plan must include installing `@tailwindcss/typography` via `@plugin "@tailwindcss/typography"` in `globals.css`. The UI spec component map (from 03-UI-SPEC.md) provides exact per-element class specifications that make this the correct approach: prose provides reset/spacing defaults; the component map's explicit Tailwind classes override color, font, and decoration.

Four files contain PT stubs that import from (or inline) `block-renderer.tsx` and need updating: `info-section.tsx`, `image-with-text.tsx`, `cta-features-section.tsx`, and `cta-feature.tsx`. The first three are actively rendered; `cta-feature.tsx` is not imported by any layout component but contains a stub and should be updated for consistency.

**Primary recommendation:** Install `@tailwindcss/typography`, rename `block-renderer.tsx` to `portable-text-renderer.tsx`, implement the full component map per 03-UI-SPEC.md, then update the 3 active layout consumers.

---

## Standard Stack

### Core

| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| `@portabletext/react` | 6.0.3 | Renders Portable Text block arrays as React JSX | Official Sanity ecosystem library; used universally for PT rendering in React |
| `@tailwindcss/typography` | 0.5.19 | Provides `prose` utility classes for prose spacing/reset in Tailwind v4 | Required because TW v4 does not include prose built-in; plugin adds the `prose` class the existing stubs already reference |

### Supporting

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| `tailwindcss` | ^4.2.1 (already installed) | Utility CSS framework | Already in project; `@tailwindcss/typography` plugs into this |
| `react` | ^18.3.1 (already installed) | UI rendering | `@portabletext/react` requires React 18 or 19 — project uses 18.3.1, meets peer dep |

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| `@portabletext/react` | `@sanity/block-content-to-react` | Legacy package, deprecated by Sanity in favor of `@portabletext/react` |
| `@tailwindcss/typography` plugin | Manual `@utility prose { ... }` in globals.css | More control, more work; typography plugin is the Tailwind-endorsed solution for v4 |
| `@tailwindcss/typography` plugin | Skip prose, use only inline Tailwind classes per element | Valid approach — UI spec component map already specifies all classes explicitly; difference is prose provides a reset for nested `<a>` and list colors that complements the explicit overrides |

**Installation (new packages only):**
```bash
cd frontend && npm install @portabletext/react
npm install --save-dev @tailwindcss/typography
```

**CSS activation (add to globals.css):**
```css
@import "tailwindcss";
@plugin "@tailwindcss/typography";
```

**Version verification:** Confirmed against npm registry on 2026-03-26.
- `@portabletext/react`: 6.0.3 (peer dep: React 18 or 19)
- `@tailwindcss/typography`: 0.5.19

---

## Architecture Patterns

### File Location and Export

```
frontend/src/components/ui/
├── portable-text-renderer.tsx   ← renamed from block-renderer.tsx
├── sanity-image.tsx
├── logo.tsx
└── ...other ui/ components
```

The file is renamed (not created alongside) so all existing imports break explicitly and can be updated in the same pass. No intermediate file or re-export needed.

### Pattern 1: PortableText Component Map

**What:** `@portabletext/react` accepts a `components` prop with keys for `block`, `marks`, `list`, and `listItem`. Each key maps a PT block type or mark type to a React component.

**When to use:** Whenever a PT block type or mark needs non-default rendering.

**Example (verified API from official README):**
```typescript
// Source: https://github.com/portabletext/react-portabletext/blob/main/README.md
import { PortableText, PortableTextComponents } from '@portabletext/react'
import type { TPortableTextBlock } from '@/types'

const components: PortableTextComponents = {
  block: {
    normal: ({ children }) => (
      <p className="mb-4 text-base font-normal leading-relaxed">{children}</p>
    ),
    h2: ({ children }) => (
      <h2 className="text-2xl font-bold font-serif text-brand-red mb-3 mt-6">
        {children}
      </h2>
    ),
    h3: ({ children }) => (
      <h3 className="text-lg font-bold font-serif text-brand-red mb-2 mt-4">
        {children}
      </h3>
    ),
  },
  marks: {
    strong: ({ children }) => <strong className="font-bold">{children}</strong>,
    em: ({ children }) => <em className="italic">{children}</em>,
    link: ({ value, children }) => {
      const target = value?.blank ? '_blank' : undefined
      return (
        <a
          href={value?.href}
          target={target}
          rel={target === '_blank' ? 'noopener noreferrer' : undefined}
          className="text-brand-red underline underline-offset-4 hover:text-brand-dark-red focus-visible:outline-brand-red"
        >
          {children}
        </a>
      )
    },
  },
  list: {
    bullet: ({ children }) => (
      <ul className="list-disc list-outside pl-6 mb-4 space-y-1">{children}</ul>
    ),
    number: ({ children }) => (
      <ol className="list-decimal list-outside pl-6 mb-4 space-y-1">{children}</ol>
    ),
  },
  listItem: {
    bullet: ({ children }) => <li className="marker:text-brand-red">{children}</li>,
    number: ({ children }) => <li className="marker:text-brand-red">{children}</li>,
  },
}

interface PortableTextRendererProps {
  content: TPortableTextBlock[]
}

export function PortableTextRenderer({ content }: PortableTextRendererProps) {
  if (!content || !Array.isArray(content) || content.length === 0) {
    return null
  }

  return <PortableText value={content} components={components} />
}
```

### Pattern 2: Consumer Import Update

**What:** Each layout component drops its inline PT stub and replaces it with `<PortableTextRenderer content={...} />`. The parent wrapper div stays unchanged — the renderer does not add its own max-width or padding.

**When to use:** On all 3 active layout consumers after the renderer file is created.

**Example (info-section.tsx):**
```tsx
// Before (stub):
import type { TInfoSection, TPortableTextBlock } from "@/types";
// ...
<div className="section-richtext">
  {info && info.length > 0 && (
    info.map((block: TPortableTextBlock) => (
      <p key={block._key}>
        {block.children?.map((child) => child.text).join("")}
      </p>
    ))
  )}
</div>

// After (renderer):
import { PortableTextRenderer } from "@/components/ui/portable-text-renderer";
// ...
<div className="section-richtext">
  <PortableTextRenderer content={info} />
</div>
```

### Recommended File Structure (unchanged)

```
frontend/src/components/
├── ui/
│   ├── portable-text-renderer.tsx   ← this phase
│   ├── sanity-image.tsx
│   └── ...
└── custom/
    ├── layouts/
    │   ├── info-section.tsx          ← import update
    │   ├── image-with-text.tsx       ← import update
    │   └── cta-features-section.tsx  ← import update
    └── cta-feature.tsx               ← import update (secondary)
```

### Anti-Patterns to Avoid

- **Renderer adds its own max-width or padding:** The renderer must not apply `max-w-*`, `mx-auto`, or `p-*`. Parent wrappers own that. Violating this breaks the `CtaFeaturesSection` card layout where the wrapper is `flex my-4 font-medium` (compact, no prose widths).
- **Define `components` object inside the component function:** This creates a new object reference on every render, causing unnecessary re-renders of PT children. Define `components` outside the function body (module scope) or memoize with `useMemo`.
- **Using `BlockRenderer` export name:** The name `BlockRenderer` belongs to `layouts/block-renderer.tsx`'s COMPONENT_REGISTRY dispatcher. The new component must be exported as `PortableTextRenderer` per D-03.
- **Keeping the `TPortableTextBlock` import in consumers:** Once the stub loops are removed, consumers no longer need `TPortableTextBlock` in scope. Remove to keep imports clean.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Nested mark rendering | Custom recursive mark resolver | `@portabletext/react` marks | PT marks can be nested (bold + link). Manual resolution requires walking mark stacks — the library handles this correctly. |
| Unknown mark/style fallback | `if/else` chains with `default` catch | Library's built-in `unknownMark` / `unknownBlockStyle` handlers | Library renders children as plain text for unknown marks by default — correct behavior for this schema. |
| Link blank/rel logic | Ad-hoc `target` attribute logic | `value?.blank` check in marks.link | Schema-defined `blank: boolean` maps directly to `target/_blank` + `rel` — one-liner, no abstraction needed. |
| List nesting | Manual list level tracking | `@portabletext/react` list rendering | Library handles nested list levels automatically. |

**Key insight:** `@portabletext/react` was built precisely because hand-rolled PT renderers always miss edge cases in mark stacks, nested lists, and inline block types. The library is battle-tested across the Sanity ecosystem.

---

## Common Pitfalls

### Pitfall 1: `prose` classes are inert without the typography plugin

**What goes wrong:** The existing stubs use `prose` and `prose-lg` classes (in `block-renderer.tsx` and `image-with-text.tsx`). In Tailwind CSS v4, `prose` classes do nothing unless `@tailwindcss/typography` is installed and activated with `@plugin "@tailwindcss/typography"` in the CSS file. Without it, the spacing baseline D-04 relies on is silently missing.

**Why it happens:** Tailwind v4 moved to CSS-first configuration. Built-in utilities are in the core, but `prose` was always a plugin — that hasn't changed.

**How to avoid:** Install `@tailwindcss/typography` as a devDependency and add `@plugin "@tailwindcss/typography";` to `globals.css` after the `@import "tailwindcss"` line.

**Warning signs:** `prose` class appears in JSX but elements render with no typographic spacing.

### Pitfall 2: Link mark receives wrong value type

**What goes wrong:** The PT link annotation in the schema has fields `href` (string) and `blank` (boolean). In the mark component, `value` is the entire annotation object. Accessing `value.href` fails if you accidentally destructure only `children`.

**Why it happens:** Decorators (strong, em) receive only `{children}`. Annotations (link) additionally receive `{value}` containing the annotation fields. The distinction is only visible in the type signature.

**How to avoid:** The link mark component must be `({ value, children }) => ...`. TypeScript with `PortableTextComponents` will surface this correctly.

**Warning signs:** Links render children but `href` is `undefined`.

### Pitfall 3: RT-02 rename interpretation

**What goes wrong:** REQUIREMENTS.md says RT-02 is "rename `ui/block-renderer.tsx` to `strapi-rich-text-renderer.tsx`." CONTEXT.md D-01 supersedes this: rename to `portable-text-renderer.tsx` instead (the Strapi renderer was already replaced in Phase 2, so no collision exists). Following REQUIREMENTS.md literally would create the wrong file.

**Why it happens:** CONTEXT.md was created after discussion resolved the naming decision. The requirement text predates that decision.

**How to avoid:** CONTEXT.md locked decisions override REQUIREMENTS.md wording. Use `portable-text-renderer.tsx`.

**Warning signs:** A file named `strapi-rich-text-renderer.tsx` appears during planning.

### Pitfall 4: `cta-feature.tsx` vs `cta-features-section.tsx`

**What goes wrong:** CONTEXT.md lists `cta-feature.tsx` as an integration point. The PT stub in `cta-feature.tsx` exists, but the file is not imported by any layout component — `cta-features-section.tsx` renders feature cards inline. Treating `cta-feature.tsx` as a required integration point could inflate the plan.

**Why it happens:** `cta-feature.tsx` appears to be either legacy code or a component that was superseded by the inline rendering in `cta-features-section.tsx`.

**How to avoid:** Update `cta-feature.tsx` for consistency (it has a PT stub that should be replaced), but recognize this as a secondary task. The three active consumers are `info-section.tsx`, `image-with-text.tsx`, and `cta-features-section.tsx`.

**Warning signs:** None — this is a scope clarity issue, not a breakage risk.

### Pitfall 5: Double prose wrapper in `image-with-text.tsx`

**What goes wrong:** `image-with-text.tsx` has a double-nested `<div className="prose prose-lg ...">` around the stub (lines 17 and 20). When replacing the stub with `PortableTextRenderer`, the outer wrapper should be kept and the inner one removed (along with the stub loop).

**Why it happens:** Phase 2 authored the stubs without cleanup.

**How to avoid:** During the consumer update, reduce the two nested prose divs to one, then place `<PortableTextRenderer content={perks} />` inside the single wrapper.

---

## Code Examples

### Full PortableTextRenderer (verified against @portabletext/react v6 README)

```typescript
// Source: https://github.com/portabletext/react-portabletext/blob/main/README.md
import { PortableText, type PortableTextComponents } from '@portabletext/react'
import type { TPortableTextBlock } from '@/types'

// Defined at module scope — referential stability across renders
const components: PortableTextComponents = {
  block: {
    normal: ({ children }) => (
      <p className="mb-4 text-base font-normal leading-relaxed">{children}</p>
    ),
    h2: ({ children }) => (
      <h2 className="text-2xl font-bold font-serif text-brand-red mb-3 mt-6">
        {children}
      </h2>
    ),
    h3: ({ children }) => (
      <h3 className="text-lg font-bold font-serif text-brand-red mb-2 mt-4">
        {children}
      </h3>
    ),
  },
  marks: {
    strong: ({ children }) => <strong className="font-bold">{children}</strong>,
    em: ({ children }) => <em className="italic">{children}</em>,
    link: ({ value, children }) => {
      const target = value?.blank === true ? '_blank' : undefined
      return (
        <a
          href={value?.href}
          target={target}
          rel={target ? 'noopener noreferrer' : undefined}
          className="text-brand-red underline underline-offset-4 hover:text-brand-dark-red focus-visible:outline-brand-red"
        >
          {children}
        </a>
      )
    },
  },
  list: {
    bullet: ({ children }) => (
      <ul className="list-disc list-outside pl-6 mb-4 space-y-1">{children}</ul>
    ),
    number: ({ children }) => (
      <ol className="list-decimal list-outside pl-6 mb-4 space-y-1">{children}</ol>
    ),
  },
  listItem: {
    bullet: ({ children }) => <li className="marker:text-brand-red">{children}</li>,
    number: ({ children }) => <li className="marker:text-brand-red">{children}</li>,
  },
}

interface PortableTextRendererProps {
  content: TPortableTextBlock[]
}

export function PortableTextRenderer({ content }: PortableTextRendererProps) {
  if (!content || !Array.isArray(content) || content.length === 0) {
    return null
  }
  return <PortableText value={content} components={components} />
}
```

### globals.css activation (add line 2)

```css
@import "tailwindcss";
@plugin "@tailwindcss/typography";  /* add this */
@import "tw-animate-css";
```

---

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| `@sanity/block-content-to-react` | `@portabletext/react` | ~2021 (Sanity v3 era) | Legacy package deprecated; new package has cleaner API and TypeScript-first |
| `plugins` array in `tailwind.config.js` | `@plugin "..."` in CSS file | Tailwind v4 (2025) | Config-file approach doesn't apply to TW v4 — must use CSS `@plugin` directive |
| Manual PT rendering (map + recursive walk) | `@portabletext/react` | Ongoing | Library handles mark stacks, nesting, unknown types — never hand-roll |

**Deprecated/outdated:**
- `@sanity/block-content-to-react`: Deprecated, not supported for new projects.
- `tailwind.config.js plugins: [require('@tailwindcss/typography')]`: Not valid in TW v4 projects — use `@plugin "@tailwindcss/typography"` in CSS.

---

## Open Questions

1. **Should `@tailwindcss/typography` be a dependency or devDependency?**
   - What we know: It's a build-time tool (generates CSS utilities); other TW plugins like `@tailwindcss/postcss` are in devDependencies.
   - What's unclear: The project's pattern — all current `tailwindcss`-adjacent packages (`tw-animate-css`, `@tailwindcss/postcss`) are in devDependencies.
   - Recommendation: Install as `--save-dev` to match project pattern.

2. **Does `cta-feature.tsx` need a PT renderer update?**
   - What we know: It has a PT stub but no import path from any layout component.
   - What's unclear: Whether it will be wired up in a future phase or is dead code.
   - Recommendation: Update it for consistency (one-line change), but do not spend a dedicated plan step on it — fold into the consumer update task.

---

## Environment Availability

| Dependency | Required By | Available | Version | Fallback |
|------------|------------|-----------|---------|----------|
| Node.js / npm | Package installation | ✓ | (project is running) | — |
| `@portabletext/react` | RT-01 (renderer) | ✗ (not in package.json) | — | None — must install |
| `@tailwindcss/typography` | D-04 (prose baseline) | ✗ (not in package.json) | 0.5.19 available | Could skip prose and use only explicit classes per element — viable since UI spec provides all class specs explicitly |

**Missing dependencies with no fallback:**
- `@portabletext/react` — must be installed before the renderer can be written.

**Missing dependencies with fallback:**
- `@tailwindcss/typography` — if skipped, prose classes remain inert. The UI spec component map specifies all Tailwind classes explicitly per element, so the renderer will style correctly without prose. Prose primarily adds spacing defaults and link decoration resets. The component map overrides those anyway, making this a low-risk skip — but D-04 explicitly calls for prose, so the plan should install it.

---

## Validation Architecture

### Test Framework

| Property | Value |
|----------|-------|
| Framework | Playwright 1.58.2 |
| Config file | `frontend/playwright.config.ts` |
| Quick run command | `cd frontend && npx playwright test tests/homepage.spec.ts --project=chromium` |
| Full suite command | `cd frontend && npx playwright test` |

### Phase Requirements → Test Map

| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| RT-01 | PT renderer renders without errors when mounted | smoke | `cd frontend && npx playwright test tests/homepage.spec.ts --project=chromium` | ✅ (homepage has layout blocks including info sections) |
| RT-01 | H2/H3 headings render with brand-red font-serif classes | visual/smoke | Manual inspection or new spec targeting `.text-brand-red.font-serif` | ❌ Wave 0 |
| RT-01 | Bullet lists render `<ul>` with `list-disc` | smoke | New spec targeting `ul.list-disc` in layout sections | ❌ Wave 0 |
| RT-01 | Links render with correct `target="_blank"` when `blank=true` | unit-level | Manual or new spec — requires Sanity content with link marks | ❌ Wave 0 (manual-only until content is seeded) |
| RT-02 | No import path errors after file rename | build | `cd frontend && npm run build` | ✅ (build covers all imports) |

**Note on RT-01 visual tests:** The existing Playwright specs test that pages load and `<main>` is non-empty. They do not assert on specific PT output classes. Since Sanity content must be seeded to render PT blocks in a running app, RT-01 visual assertions are best verified by build success + manual review of a running dev server.

### Sampling Rate

- **Per task commit:** `cd frontend && npm run build` — TypeScript compilation catches import errors immediately
- **Per wave merge:** `cd frontend && npx playwright test tests/homepage.spec.ts --project=chromium`
- **Phase gate:** Full build + smoke tests green before `/gsd:verify-work`

### Wave 0 Gaps

- [ ] `frontend/tests/portable-text-renderer.spec.ts` — smoke test asserting `h2.text-brand-red`, `ul.list-disc`, `ol.list-decimal` render in sections (requires dev server with seeded content, so this may be manual-only)
- No framework install needed — Playwright already installed at 1.58.2

*(If smoke test is manual-only due to content seeding requirement: note in plan — "RT-01 visual correctness verified by running `npm run dev` and inspecting info-section / image-with-text pages")*

---

## Sources

### Primary (HIGH confidence)

- `@portabletext/react` README (GitHub) — component API, marks value prop, PortableTextComponents type, `components` prop structure
- npm registry — `@portabletext/react` v6.0.3, peer deps confirmed as React 18/19
- npm registry — `@tailwindcss/typography` v0.5.19
- `frontend/package.json` — confirmed `@portabletext/react` absent; `@tailwindcss/typography` absent; React 18.3.1 present
- `frontend/src/app/globals.css` — confirmed `@tailwindcss/typography` not activated; `prose` classes not defined; brand color variables verified
- `03-UI-SPEC.md` — exact per-element Tailwind class specifications verified against existing globals.css patterns

### Secondary (MEDIUM confidence)

- GitHub Discussion tailwindlabs/tailwindcss#14120 — confirms `prose` requires `@tailwindcss/typography` plugin in TW v4 and `@plugin` CSS directive is the activation method
- WebSearch — @tailwindcss/typography v4 compatibility confirmed via multiple sources

### Tertiary (LOW confidence)

- None — all critical claims verified against primary sources.

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — package versions confirmed from npm registry; API verified from official README
- Architecture: HIGH — component map structure verified against @portabletext/react v6 docs; UI spec provides exact class specifications
- Pitfalls: HIGH — `prose` inertness confirmed by reading globals.css (no plugin) + TW v4 discussion; mark annotation `value` prop confirmed from official README

**Research date:** 2026-03-26
**Valid until:** 2026-06-26 (stable ecosystem — @portabletext/react v6 is current, Tailwind v4 plugin API is stable)
