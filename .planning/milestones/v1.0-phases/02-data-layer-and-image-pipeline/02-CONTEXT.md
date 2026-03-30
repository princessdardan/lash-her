# Phase 2: Data Layer and Image Pipeline - Context

**Gathered:** 2026-03-25
**Status:** Ready for planning

<domain>
## Phase Boundary

Replace all Strapi REST data loaders with GROQ-based loaders fetching from Sanity, update the block renderer to dispatch on Sanity `_type` keys, build a Sanity image component with hotspot/crop support, rewrite TypeScript types to match Sanity document shapes, and wire all 5 content pages (homepage, contact, gallery, training, training-programs/[slug]) to render correct content from Sanity data. Images load from cdn.sanity.io. Old Strapi data layer files are preserved (renamed) for Phase 6 cleanup.

</domain>

<decisions>
## Implementation Decisions

### GROQ Loader Organization
- **D-01:** Keep the single `loaders.ts` file pattern — one file exporting a `loaders` object with all functions. GROQ queries written inline in each function.
- **D-02:** Import the Sanity read client directly from `@/sanity/lib/client` in each loader. No wrapper function — call `client.fetch<T>(query, params)` directly. Matches next-sanity conventions.
- **D-03:** Old Strapi files kept during Phase 2: rename `loaders.ts` to `strapi-loaders.ts` and `data-api.ts` to `strapi-data-api.ts`. New Sanity `loaders.ts` created fresh. Clean removal in Phase 6.

### Image Component Design
- **D-04:** New `sanity-image.tsx` component accepts the raw Sanity image object directly (asset ref, hotspot, crop fields). Uses `@sanity/image-url` builder internally to generate URLs. Layout components pass the Sanity image field as-is — no manual URL extraction at the caller site.
- **D-05:** Full hotspot/crop support enabled. The image component reads hotspot and crop metadata from the Sanity image object and generates crop-aware URLs via the image URL builder. Editors can set focal points in Studio.
- **D-06:** Old `strapi-image.tsx` stays as-is during Phase 2. New `sanity-image.tsx` created alongside. Layout components switch to the new one. Clean removal in Phase 6.

### Block Renderer Transition
- **D-07:** Big bang switch — update `COMPONENT_REGISTRY` keys from `layout.hero-section` format to `heroSection` format all at once. Change `BaseBlock` discriminator from `__component` to `_type`, add `_key` field. No dual-registry period.
- **D-08:** Update all auxiliary sets in the same phase: `NON_RENDERABLE_COMPONENTS` keys become Sanity `_type` names (e.g., `generalInquiryLabels`), `SKIP_ANIMATION` keys become Sanity `_type` names (e.g., `heroSection`). Everything stays consistent.

### Type Definitions Strategy
- **D-09:** Keep single barrel `types/index.ts` file. Replace all Strapi-shaped types with Sanity-shaped ones. Remove `TStrapiResponse<T>` wrapper entirely — GROQ returns plain objects directly.
- **D-10:** Remove all Strapi-specific types in this phase (TStrapiResponse, TImage with Strapi shape, Strapi rich text types like BlocksContent). Clean break — nothing references them after loaders switch to GROQ.
- **D-11:** Move all block union types (TBlocks, TContactPageBlocks, TGalleryPageBlocks, etc.) into `types/index.ts`. Pages import from `@/types`. Eliminates the current circular import pattern where `types/index.ts` imports from page files.

### Claude's Discretion
- Exact GROQ query projections for each loader (field selection, nested object expansion)
- Sanity image URL builder configuration (default quality, format settings)
- Internal structure of new Sanity TypeScript types (field names follow schema, specific optional/required decisions)
- `getBlockKey` function update to use `_key` instead of `documentId`/`id`
- Whether to keep `getStrapiURL()` and `getStrapiMedia()` utilities or just leave them in the renamed files

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Current Data Layer (being replaced)
- `frontend/src/data/loaders.ts` — All 9 Strapi loader functions showing which fields each page needs populated
- `frontend/src/data/data-api.ts` — Current Strapi API client (will be renamed, not modified)
- `frontend/src/types/index.ts` — Current type definitions (will be rewritten for Sanity shapes)

### Block Renderer (being updated)
- `frontend/src/components/custom/layouts/block-renderer.tsx` — COMPONENT_REGISTRY, NON_RENDERABLE_COMPONENTS, SKIP_ANIMATION, BaseBlock interface, getBlockKey function

### Image Component (being replaced)
- `frontend/src/components/ui/strapi-image.tsx` — Current StrapiImage component and getStrapiMedia utility

### Sanity Infrastructure (from Phase 1)
- `frontend/src/sanity/lib/client.ts` — Read client configuration (useCdn: true)
- `frontend/src/sanity/lib/write-client.ts` — Write client (not needed in this phase)
- `frontend/src/sanity/env.ts` — Environment variable configuration
- `frontend/src/sanity/schemas/` — All schema definitions (documents/ and objects/) defining field names and structures

### Page Files (consuming the data layer)
- `frontend/src/app/(site)/page.tsx` — Homepage
- `frontend/src/app/(site)/contact/page.tsx` — Contact page
- `frontend/src/app/(site)/gallery/page.tsx` — Gallery page
- `frontend/src/app/(site)/training/page.tsx` — Training page
- `frontend/src/app/(site)/training-programs/[slug]/page.tsx` — Dynamic training program pages
- `frontend/src/app/(site)/layout.tsx` — Site layout (global data + menu caching)

### Configuration
- `frontend/next.config.ts` — Image remote patterns (needs cdn.sanity.io added)

### Project Context
- `.planning/REQUIREMENTS.md` — DATA-01 through DATA-05, IMG-01 through IMG-04
- `.planning/ROADMAP.md` — Phase 2 success criteria
- `.planning/phases/01-schema-studio-and-infrastructure/01-CONTEXT.md` — Phase 1 decisions (schema naming, file organization)

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- `frontend/src/sanity/lib/client.ts` — Read client already configured with useCdn: true, ready for GROQ fetches
- `frontend/src/sanity/schemas/` — All 10 document schemas and 22 object schemas defining exact field names for GROQ projections
- `frontend/src/lib/utils.ts` — `cn()` utility for className merging in new image component
- `frontend/src/components/custom/layouts/block-animation-wrapper.tsx` — Animation wrapper unchanged, works with any block key format
- `frontend/src/components/custom/layouts/block-error-boundary.tsx` — Error boundary unchanged, componentName just switches to _type format

### Established Patterns
- **Single loaders file**: `loaders.ts` exports object with all fetch functions — keeping this pattern
- **Single types barrel**: `types/index.ts` exports all types — keeping this pattern
- **Block renderer registry**: `COMPONENT_REGISTRY` maps CMS keys to React components — same pattern, different keys
- **unstable_cache in layout.tsx**: Global/menu data cached with tags — will need GROQ versions of these loaders
- **ISR revalidate = 1800**: All pages export this — unchanged, still applies to Sanity-backed pages

### Integration Points
- `frontend/src/app/(site)/layout.tsx` — Must switch from `loaders.getGlobalData()` and `loaders.getMainMenuData()` to new Sanity loaders. The `unstable_cache` wrapper and `validateApiResponse` call patterns will change.
- `frontend/src/lib/error-handler.ts` — `validateApiResponse()` is built around `TStrapiResponse<T>` — pages will need updated error handling since GROQ returns plain objects
- `frontend/next.config.ts` — `remotePatterns` needs `cdn.sanity.io` added; Strapi patterns kept until Phase 6
- All 14 layout components — Currently receive data via `data` prop typed to Strapi shapes. Props may need updating if Sanity field names differ from Strapi field names.

</code_context>

<specifics>
## Specific Ideas

No specific requirements — open to standard approaches following next-sanity and @sanity/image-url conventions.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope.

</deferred>

---

*Phase: 02-data-layer-and-image-pipeline*
*Context gathered: 2026-03-25*
