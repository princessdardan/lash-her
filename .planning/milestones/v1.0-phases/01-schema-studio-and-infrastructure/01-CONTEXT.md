# Phase 1: Schema, Studio, and Infrastructure - Context

**Gathered:** 2026-03-24
**Status:** Ready for planning

<domain>
## Phase Boundary

Define all Sanity schemas mirroring the existing Strapi content model, embed Sanity Studio at /studio with a structured sidebar, and configure read/write clients. This phase produces the foundation every subsequent phase builds on — no frontend changes, no data migration, no GROQ queries yet.

</domain>

<decisions>
## Implementation Decisions

### Studio Editing Experience
- **D-01:** Sidebar organized into 3 sections: **Pages** (all 7 singletons), **Content** (training programs collection), **Submissions** (general inquiries + contact forms)
- **D-02:** Singletons open directly when clicked — no intermediate document list view. Use Sanity's `documentListItem().child()` pattern.
- **D-03:** Friendly document titles in sidebar: "Homepage", "Contact Page", "Gallery", "Training", "Training Programs Overview", "Global Settings", "Navigation Menu"
- **D-04:** Block array items in Studio show compact type list with key field preview (e.g., "Hero Section — Welcome to Lash Her"), not expanded field previews

### Schema File Organization
- **D-05:** All Sanity code lives under `frontend/src/sanity/` — schemas, client config, Studio structure, and `sanity.config.ts`
- **D-06:** Schemas grouped into `schemas/documents/` (10 document types) and `schemas/objects/` (14 layout block types + 6 reusable sub-objects), with a barrel `schemas/index.ts`
- **D-07:** Schema files use kebab-case filenames matching the existing frontend convention (e.g., `hero-section.ts`, `home-page.ts`)

### Block Array Design
- **D-08:** Each page document defines its own allowed block types in its `blocks` array field — per-page type restrictions, not a shared global set. Mirrors how Strapi's dynamic zones are configured per content type.
- **D-09:** Block array field named `blocks` on all page documents — same as Strapi. Minimizes frontend changes in Phase 2.

### Content Model Fidelity
- **D-10:** Mirror Strapi's structure 1:1 — `trainingPage` and `trainingProgramsPage` remain separate singleton documents mapping to `/training` and `/training-programs` routes
- **D-11:** Skip the unused `section` collection type — no Sanity schema for it (already out-of-scope in REQUIREMENTS.md)
- **D-12:** Clean up ambiguous type names during migration. Layout component `contact-form` (holds form labels) becomes `contactFormLabels` to match the React component and avoid collision with the `contactForm` submission document type. `general-inquiry-labels` becomes `generalInquiryLabels`.
- **D-13:** Rich text fields defined as Portable Text (`block` type) from day one in the schema, even though the renderer is built in Phase 3. No temporary placeholders.

### Claude's Discretion
- Icon choices for sidebar document types
- Exact Portable Text block configuration (which decorators, annotations, styles to enable)
- Sanity `preview` configuration details for each schema type
- Internal structure of reusable sub-objects (field ordering, validation rules)

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Strapi Content Model (source of truth for schema mirroring)
- `backend/src/api/home-page/content-types/home-page/schema.json` — Homepage document structure and dynamic zone config
- `backend/src/api/contact/content-types/contact/schema.json` — Contact page structure
- `backend/src/api/gallery/content-types/gallery/schema.json` — Gallery page structure
- `backend/src/api/training/content-types/training/schema.json` — Training page structure
- `backend/src/api/training-programs-page/content-types/training-programs-page/schema.json` — Training programs overview page
- `backend/src/api/training-program/content-types/training-program/schema.json` — Training program collection type
- `backend/src/api/global/content-types/global/schema.json` — Global settings
- `backend/src/api/main-menu/content-types/main-menu/schema.json` — Navigation menu with polymorphic items
- `backend/src/api/contact-form/content-types/contact-form/schema.json` — Contact form submissions
- `backend/src/api/general-inquiry/content-types/general-inquiry/schema.json` — General inquiry submissions
- `backend/src/components/layout/` — All 14 layout component schemas (hero-section, features-section, etc.)
- `backend/src/components/components/` — All 6 reusable sub-component schemas (link, feature, etc.)
- `backend/src/components/menu/` — Menu component schemas (dropdown, menu-link)

### Frontend Integration Points
- `frontend/src/components/custom/layouts/block-renderer.tsx` — COMPONENT_REGISTRY mapping (keys will change from `layout.hero-section` to `heroSection`)
- `frontend/src/types/index.ts` — Current type definitions that Phase 2 will rewrite
- `frontend/src/data/loaders.ts` — Current data loaders showing which fields each page needs populated

### Project Context
- `.planning/REQUIREMENTS.md` — SCHEMA-01 through SCHEMA-05, INFRA-01 through INFRA-04
- `.planning/ROADMAP.md` — Phase 1 success criteria

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- `frontend/src/lib/utils.ts` — `cn()` utility, can be used in any new Sanity components
- `frontend/src/components/ui/` — shadcn/ui primitives available for Studio customization if needed
- `frontend/next.config.ts` — Already has remote patterns config that will need `cdn.sanity.io` added

### Established Patterns
- **Kebab-case files, PascalCase exports** — Schema files should follow this (e.g., `hero-section.ts` exports `heroSection`)
- **Named exports** preferred over defaults — Schema files should use `export const heroSection = defineType(...)`
- **Barrel files** — `schemas/index.ts` should re-export all schemas as an array for `sanity.config.ts`
- **`@/` path alias** — Maps to `frontend/src/*`, so Sanity imports will be `@/sanity/...`

### Integration Points
- `frontend/src/app/studio/[[...tool]]/page.tsx` — Studio route (to be created)
- `frontend/src/sanity/sanity.config.ts` — Main Sanity config (to be created)
- `frontend/src/sanity/lib/client.ts` — Read/write client (to be created)
- `frontend/.env.local` — Needs SANITY_PROJECT_ID, SANITY_DATASET, SANITY_WRITE_TOKEN, SANITY_API_VERSION

</code_context>

<specifics>
## Specific Ideas

No specific requirements — open to standard approaches for Sanity schema implementation following `next-sanity` conventions.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope.

</deferred>

---

*Phase: 01-schema-studio-and-infrastructure*
*Context gathered: 2026-03-24*
