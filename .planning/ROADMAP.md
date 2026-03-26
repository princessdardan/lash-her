# Roadmap: Lash Her — Strapi to Sanity Migration

## Overview

Six phases migrate the Lash Her website from Strapi 5 to Sanity as the content platform. The build order is dependency-constrained: schema must exist before queries can be written, queries before types can be inferred, types before pages compile. Forms and rich text are independent tracks that converge with the data layer. Migration runs last, after all code is verified against a staging dataset, followed by Strapi removal.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [x] **Phase 1: Schema, Studio, and Infrastructure** - Define all Sanity schemas, embed Studio at /studio, and wire up read/write clients
- [ ] **Phase 2: Data Layer and Image Pipeline** - Replace all Strapi REST loaders with GROQ, update block renderer, build Sanity image component, wire all content pages
- [ ] **Phase 3: Rich Text and Portable Text Renderer** - Build brand-styled PortableText renderer, rename legacy rich text component
- [ ] **Phase 4: Forms and Email** - Replace Strapi lifecycle hooks with Server Actions writing to Sanity + calling Resend directly
- [ ] **Phase 5: Cache Revalidation** - Wire on-demand ISR via Sanity webhook calling revalidateTag
- [ ] **Phase 6: Content Migration and Cleanup** - Run migration scripts against staging then production, verify, and remove Strapi

## Phase Details

### Phase 1: Schema, Studio, and Infrastructure
**Goal**: The Sanity content model exists in code, Studio is accessible at /studio and reflects all document types, and both read and write clients are configured — the foundation every other phase builds on
**Depends on**: Nothing (first phase)
**Requirements**: SCHEMA-01, SCHEMA-02, SCHEMA-03, SCHEMA-04, SCHEMA-05, INFRA-01, INFRA-02, INFRA-03, INFRA-04
**Success Criteria** (what must be TRUE):
  1. Navigating to /studio in the browser loads the Sanity Studio with all 7 singleton documents and the training-program collection visible in the sidebar
  2. Every layout block type (heroSection, featuresSection, imageWithText, etc.) appears as an object option when editing dynamic zone content in Studio
  3. The navigation menu document accepts both direct link items and multi-section dropdown items in the same array field
  4. A GROQ query run against the staging dataset returns structured data matching the schema field names
  5. The Sanity read client resolves without error in a Next.js Server Component; the write client is importable only in server-only modules
**Plans**: 4 plans

Plans:
- [x] 01-01-PLAN.md — Install Sanity packages, environment config, read/write clients
- [x] 01-02-PLAN.md — Define all object schemas (sub-objects, navigation, layout blocks)
- [x] 01-03-PLAN.md — Define all document schemas and schema barrel file
- [x] 01-04-PLAN.md — Studio config, structure builder, and /studio route

### Phase 2: Data Layer and Image Pipeline
**Goal**: All five content pages (homepage, contact, gallery, training, training programs) render correct content from Sanity data, images load from cdn.sanity.io, and the block renderer dispatches on Sanity _type keys
**Depends on**: Phase 1
**Requirements**: DATA-01, DATA-02, DATA-03, DATA-04, DATA-05, IMG-01, IMG-02, IMG-03, IMG-04
**Success Criteria** (what must be TRUE):
  1. Every page route (/, /contact, /gallery, /training, /training-programs/[slug]) renders without TypeScript errors and displays content pulled from the staging Sanity dataset
  2. Images on all pages load from cdn.sanity.io with correct dimensions and no broken image placeholders
  3. The block renderer correctly renders all 14 layout block types — no blank blocks, no console errors about unrecognized _type values
  4. /training-programs/[slug] generates static params from a GROQ query (not a hardcoded list) and each slug resolves to the correct program
  5. TypeScript compilation passes with strict mode — no implicit any, no TStrapiResponse references remaining
**Plans**: 4 plans
**UI hint**: yes

Plans:
- [x] 02-01-PLAN.md — Rename Strapi files, rewrite types for Sanity shapes, create SanityImage component, add CDN pattern
- [ ] 02-02-PLAN.md — Create GROQ-based loaders and update block renderer registry keys
- [ ] 02-03-PLAN.md — Update all layout components and main menu for Sanity prop types
- [ ] 02-04-PLAN.md — Wire all pages and site layout to Sanity loaders

### Phase 3: Rich Text and Portable Text Renderer
**Goal**: All rich-text content in the site (info sections, training program descriptions, any body copy) renders with correct brand styling using the new Portable Text renderer, with no naming collision against the legacy renderer
**Depends on**: Phase 1
**Requirements**: RT-01, RT-02
**Success Criteria** (what must be TRUE):
  1. The legacy rich text component is renamed to strapi-rich-text-renderer.tsx and no import paths break
  2. Rich text content (headings, paragraphs, bold, italic, lists, links) in any layout block renders visually identically to the current Strapi-backed site
  3. The PortableTextRenderer component is importable and renders without errors when given a Portable Text array from Sanity
**Plans**: TBD
**UI hint**: yes

### Phase 4: Forms and Email
**Goal**: Both contact forms submit successfully, store the submission as a Sanity document, and trigger Resend admin notification and user confirmation emails — with no dependency on the Strapi backend
**Depends on**: Phase 1
**Requirements**: FORM-01, FORM-02, FORM-03, FORM-04
**Success Criteria** (what must be TRUE):
  1. Submitting the general inquiry form creates a new document in the Sanity generalInquiry collection visible in Studio
  2. Submitting the training contact form creates a new document in the Sanity contactForm collection visible in Studio
  3. Both form submissions trigger admin notification and user confirmation emails via Resend within seconds of submission
  4. The Sanity write token used by form Server Actions is scoped to create-only on contactForm and generalInquiry document types
**Plans**: TBD

### Phase 5: Cache Revalidation
**Goal**: Publishing any document in Sanity Studio causes the corresponding page to reflect updated content within seconds, without waiting for the 30-minute ISR polling interval
**Depends on**: Phase 2
**Requirements**: INFRA-05, INFRA-06
**Success Criteria** (what must be TRUE):
  1. Editing and publishing a document in Sanity Studio triggers a POST to /api/revalidate within seconds (visible in Vercel function logs)
  2. The revalidate endpoint rejects requests with an invalid or missing webhook signature and returns 401
  3. After a valid webhook fires, a hard refresh of the corresponding page shows the updated content without waiting for ISR interval expiry
**Plans**: TBD

### Phase 6: Content Migration and Cleanup
**Goal**: All existing Strapi content exists in Sanity as published documents, the site runs entirely on Sanity, and the Strapi backend and all Strapi-specific code is removed
**Depends on**: Phase 2, Phase 3, Phase 4, Phase 5
**Requirements**: MIG-01, MIG-02, MIG-03, MIG-04, MIG-05, MIG-06, CLEAN-01, CLEAN-02, CLEAN-03, CLEAN-04, CLEAN-05
**Success Criteria** (what must be TRUE):
  1. Running the migration script against the staging dataset produces document counts matching Strapi (all singleton pages, all training programs, all form submissions present as published documents)
  2. All pages on the staging site render real migrated content — including images served from cdn.sanity.io and rich text rendered as Portable Text
  3. The production site renders identically to the current Strapi-backed site after cutover, with zero broken pages or missing images
  4. The backend/ directory is deleted and the npm workspaces config no longer references it — `npm install` from the root succeeds
  5. No Strapi environment variables, import paths, or package references remain in the frontend codebase
**Plans**: TBD

## Progress

**Execution Order:**
Phases execute in order: 1 → 2 → 3 → 4 → 5 → 6 (Phases 3 and 4 can run parallel with Phase 2 development but integrate after Phase 1)

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Schema, Studio, and Infrastructure | 4/4 | Complete | 2026-03-25 |
| 2. Data Layer and Image Pipeline | 0/4 | Not started | - |
| 3. Rich Text and Portable Text Renderer | 0/? | Not started | - |
| 4. Forms and Email | 0/? | Not started | - |
| 5. Cache Revalidation | 0/? | Not started | - |
| 6. Content Migration and Cleanup | 0/? | Not started | - |
