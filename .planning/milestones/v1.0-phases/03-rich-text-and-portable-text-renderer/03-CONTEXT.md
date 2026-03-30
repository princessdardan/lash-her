# Phase 3: Rich Text and Portable Text Renderer - Context

**Gathered:** 2026-03-26
**Status:** Ready for planning

<domain>
## Phase Boundary

Build a brand-styled Portable Text renderer using `@portabletext/react` with a component map matching existing visual output (headings, paragraphs, lists, links, bold, italic). Replace the current plain-text stub in `ui/block-renderer.tsx` with the full renderer, rename the file to `portable-text-renderer.tsx`, and update all 3 layout components that consume rich text to use the new `PortableTextRenderer` export. No naming collision remains after Phase 2 replaced the original Strapi renderer with a PT stub — RT-02's literal rename to `strapi-rich-text-renderer.tsx` is superseded by the in-place replacement.

</domain>

<decisions>
## Implementation Decisions

### Renderer Naming & File Organization
- **D-01:** Replace `ui/block-renderer.tsx` in-place with the full `@portabletext/react` renderer. Rename file to `portable-text-renderer.tsx`. No `strapi-rich-text-renderer.tsx` needed — the original Strapi renderer was already replaced by a PT stub in Phase 2.
- **D-02:** File location: `frontend/src/components/ui/portable-text-renderer.tsx` — stays in `ui/` consistent with other reusable UI components (sanity-image.tsx, logo.tsx).
- **D-03:** Export name: `PortableTextRenderer` — clear, descriptive, no collision with `layouts/block-renderer.tsx`'s `BlockRenderer` export.

### Brand Styling Approach
- **D-04:** Use Tailwind Typography `prose` classes as the base, then override specific elements (h2, h3, links, lists) in the `@portabletext/react` component map with brand classes (font-serif, text-brand-red, etc.). Prose handles spacing/defaults, component map handles brand look.
- **D-05:** Links honor the schema's `blank` field: open in new tab when `blank=true` (`target="_blank"` + `rel="noopener noreferrer"`), same tab otherwise. Matches schema intent and gives editors control.

### Context-Specific Rendering
- **D-06:** One universal renderer — `PortableTextRenderer` renders the same styled PT output everywhere. Parent layout components (info-section, image-with-text, cta-features-section) wrap it in their own container divs with context-specific classes (`.section-richtext`, `prose-lg`, etc.).
- **D-07:** Layout components pass the full Portable Text array: `<PortableTextRenderer content={data.info} />`. The renderer handles the full array internally. No per-block mapping in parent components.

### Claude's Discretion
- Exact Tailwind prose modifier classes and brand overrides in the component map
- Whether to install `@tailwindcss/typography` if not already present, or rely on existing prose classes
- Internal structure of the component map (block, marks, list, listItem handlers)
- How to handle edge cases (empty arrays, malformed blocks)

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Current PT Stub (being replaced)
- `frontend/src/components/ui/block-renderer.tsx` — Current plain-text stub renderer. Will be replaced and renamed to `portable-text-renderer.tsx`.

### Portable Text Schema Configuration
- `frontend/src/sanity/schemas/objects/layout/info-section.ts` — PT block config: styles [normal, h2, h3], lists [bullet, number], marks [strong, em], annotations [link with href + blank]
- `frontend/src/sanity/schemas/objects/layout/image-with-text.ts` — Same PT block config on `perks` field
- `frontend/src/sanity/schemas/objects/shared/cta-feature.ts` — Same PT block config on `features` field

### Layout Components Consuming Rich Text
- `frontend/src/components/custom/layouts/info-section.tsx` — Uses `.section-richtext` wrapper, inline PT stub
- `frontend/src/components/custom/layouts/image-with-text.tsx` — Uses `prose prose-lg` wrapper, inline PT stub
- `frontend/src/components/custom/layouts/cta-features-section.tsx` — Inline PT stub in feature cards
- `frontend/src/components/custom/cta-feature.tsx` — Individual CTA feature card with PT stub

### Type Definitions
- `frontend/src/types/index.ts` — `TPortableTextBlock` interface (lines 11-19) already defined with _type, _key, style, children, markDefs, listItem, level

### Brand Styling Reference
- `frontend/src/app/globals.css` — `.section-richtext` class (line 334), card heading classes, hero heading classes — defines existing brand visual patterns

### Project Context
- `.planning/REQUIREMENTS.md` — RT-01, RT-02
- `.planning/ROADMAP.md` — Phase 3 success criteria
- `.planning/phases/01-schema-studio-and-infrastructure/01-CONTEXT.md` — D-13: PT block config decisions
- `.planning/phases/02-data-layer-and-image-pipeline/02-CONTEXT.md` — D-07/D-08: block renderer transition decisions

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- `TPortableTextBlock` type already defined in `types/index.ts` — ready for use in renderer props
- Tailwind `prose` classes already used in image-with-text and block-renderer stub — pattern established
- `.section-richtext` CSS class in globals.css — card-style wrapper for info sections
- Brand color classes: `text-brand-red`, `font-serif` — used throughout for headings

### Established Patterns
- **Single component per file** — ui/ components follow this pattern
- **Named exports** — all components use named exports, not defaults
- **Tailwind for styling** — no CSS modules, no styled-components
- **@/ path alias** — imports use `@/components/ui/...`

### Integration Points
- 3 layout components need import updates: info-section.tsx, image-with-text.tsx, cta-features-section.tsx
- 1 additional component: cta-feature.tsx (individual card)
- `@portabletext/react` package needs to be installed (not currently in package.json)
- No other files import from `ui/block-renderer.tsx` beyond the 3 layout components + cta-feature

</code_context>

<specifics>
## Specific Ideas

No specific requirements — open to standard `@portabletext/react` patterns with brand styling matching the existing site output.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope.

</deferred>

---

*Phase: 03-rich-text-and-portable-text-renderer*
*Context gathered: 2026-03-26*
