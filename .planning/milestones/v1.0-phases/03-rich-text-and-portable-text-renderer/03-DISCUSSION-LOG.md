# Phase 3: Rich Text and Portable Text Renderer - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-03-26
**Phase:** 03-rich-text-and-portable-text-renderer
**Areas discussed:** Renderer naming & file org, Brand styling approach, Context-specific rendering

---

## Renderer Naming & File Organization

### File handling strategy

| Option | Description | Selected |
|--------|-------------|----------|
| Replace in-place (Recommended) | Replace ui/block-renderer.tsx content with full @portabletext/react renderer, rename to portable-text-renderer.tsx. No strapi-rich-text-renderer.tsx needed. | ✓ |
| New file alongside | Create new portable-text-renderer.tsx in ui/, keep ui/block-renderer.tsx as-is until Phase 6 cleanup. | |
| Follow RT-02 literally | Rename ui/block-renderer.tsx to strapi-rich-text-renderer.tsx, then create new portable-text-renderer.tsx. | |

**User's choice:** Replace in-place (Recommended)
**Notes:** The original Strapi renderer was already replaced by a PT stub in Phase 2, making the RT-02 literal rename unnecessary.

### File location

| Option | Description | Selected |
|--------|-------------|----------|
| ui/portable-text-renderer.tsx (Recommended) | Stays in ui/ consistent with other reusable UI components. | ✓ |
| custom/portable-text-renderer.tsx | Under custom/ alongside layout components. | |
| lib/portable-text.tsx | Under lib/ as a utility-style module. | |

**User's choice:** ui/portable-text-renderer.tsx (Recommended)

### Export name

| Option | Description | Selected |
|--------|-------------|----------|
| PortableTextRenderer (Recommended) | Clear, descriptive, no collision with layouts BlockRenderer. | ✓ |
| RichText | Short and ergonomic. | |
| You decide | Claude picks based on conventions. | |

**User's choice:** PortableTextRenderer (Recommended)

---

## Brand Styling Approach

### Styling strategy

| Option | Description | Selected |
|--------|-------------|----------|
| Tailwind prose + component map (Recommended) | Use @tailwindcss/typography prose as base, override specific elements in component map with brand classes. | ✓ |
| Pure component map styling | No prose plugin. Every element gets explicit Tailwind classes. | |
| CSS-only approach | Render default HTML, style via globals.css selectors. | |
| You decide | Claude picks the approach matching existing patterns. | |

**User's choice:** Tailwind prose + component map (Recommended)

### Link behavior

| Option | Description | Selected |
|--------|-------------|----------|
| Honor blank field (Recommended) | Links open in new tab when blank=true, same tab otherwise. | ✓ |
| All external = new tab | Auto-detect external links and open in new tab regardless of blank field. | |
| You decide | Claude picks most standard approach. | |

**User's choice:** Honor blank field (Recommended)

---

## Context-Specific Rendering

### Rendering approach

| Option | Description | Selected |
|--------|-------------|----------|
| One renderer, parent controls wrapper (Recommended) | PortableTextRenderer renders same output everywhere. Parents wrap with context-specific classes. | ✓ |
| Renderer accepts className prop | Optional className prop for context-specific styling. | |
| Variant-based rendering | Variant prop switches between different component maps. | |

**User's choice:** One renderer, parent controls wrapper (Recommended)

### Integration pattern

| Option | Description | Selected |
|--------|-------------|----------|
| Pass full array (Recommended) | <PortableTextRenderer content={data.info} /> — renderer handles full PT array. | ✓ |
| Map and pass individually | Layout components keep .map() and pass each block individually. | |
| You decide | Claude picks based on @portabletext/react best practices. | |

**User's choice:** Pass full array (Recommended)

---

## Claude's Discretion

- Exact Tailwind prose modifier classes and brand overrides
- @tailwindcss/typography installation if needed
- Component map internal structure
- Edge case handling

## Deferred Ideas

None — discussion stayed within phase scope.
