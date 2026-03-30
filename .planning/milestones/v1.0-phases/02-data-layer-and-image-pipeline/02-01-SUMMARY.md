---
phase: 02-data-layer-and-image-pipeline
plan: 01
subsystem: api
tags: [sanity, typescript, next-image, image-url, types, migration]

# Dependency graph
requires:
  - phase: 01-sanity-studio-and-schema
    provides: Sanity client at frontend/src/sanity/lib/client.ts and schema field definitions

provides:
  - Sanity-shaped TypeScript types with _type/_key discriminators (frontend/src/types/index.ts)
  - SanityImage component with hotspot/crop support via @sanity/image-url (frontend/src/components/ui/sanity-image.tsx)
  - cdn.sanity.io remote pattern in next.config.ts
  - Preserved Strapi files as strapi-loaders.ts and strapi-data-api.ts for Phase 6 cleanup

affects:
  - 02-data-layer-and-image-pipeline
  - 03-block-renderer-and-layout-components
  - 04-forms-and-email
  - 05-content-migration

# Tech tracking
tech-stack:
  added: ["@sanity/image-url (root node_modules, already present from Phase 1)"]
  patterns:
    - "imageUrlBuilder initialized at module level (not per-render) for performance"
    - "TSanityImage uses asset._ref pattern instead of url string"
    - "TPortableTextBlock for all rich text fields (perks, info, features)"
    - "_type/_key on all block interfaces for Sanity array discriminators"
    - "Block union type TLayoutBlock defined in types/index.ts — no circular page imports"

key-files:
  created:
    - frontend/src/components/ui/sanity-image.tsx
    - frontend/src/data/strapi-loaders.ts
    - frontend/src/data/strapi-data-api.ts
  modified:
    - frontend/src/types/index.ts
    - frontend/next.config.ts

key-decisions:
  - "All Strapi block types replaced: _type/_key instead of __component/id discriminators"
  - "TLayoutBlock union moved to types/index.ts to eliminate circular imports from page files"
  - "Strapi loaders and API client renamed (not deleted) — preserved for Phase 6 cleanup"
  - "SanityImage builds URL via @sanity/image-url — hotspot/crop read automatically from image object"
  - "cdn.sanity.io added as FIRST remote pattern, Strapi patterns kept until Phase 6"

patterns-established:
  - "SanityImage: import { SanityImage } from @/components/ui/sanity-image — passes raw TSanityImage object"
  - "Type imports: import type { TSanityImage, TLayoutBlock, ... } from @/types"
  - "Block discrimination: switch on block._type — matches Sanity camelCase schema types"

requirements-completed: [DATA-03, IMG-01, IMG-03, IMG-04]

# Metrics
duration: 2min
completed: 2026-03-26
---

# Phase 2 Plan 01: Data Layer Foundation Summary

**Sanity type foundation with TSanityImage, TPortableTextBlock, TLayoutBlock union, SanityImage component using @sanity/image-url, and cdn.sanity.io remote pattern**

## Performance

- **Duration:** 2 min
- **Started:** 2026-03-26T01:42:39Z
- **Completed:** 2026-03-26T01:43:43Z
- **Tasks:** 2
- **Files modified:** 5

## Accomplishments

- All TypeScript types rewritten from Strapi shapes to Sanity _type/_key discriminators — eliminates all `documentId`, `createdAt`, `__component` Strapi fields
- SanityImage component created with @sanity/image-url builder for hotspot/crop-aware image rendering and auto WebP/AVIF format
- cdn.sanity.io added to next.config.ts remotePatterns as first entry; all existing Strapi patterns preserved for Phase 6

## Task Commits

Each task was committed atomically:

1. **Task 1: Rename Strapi files and rewrite types/index.ts** - `4807c7b` (feat)
2. **Task 2: Create SanityImage component and add cdn.sanity.io remote pattern** - `ac6488e` (feat)

## Files Created/Modified

- `frontend/src/types/index.ts` - Complete rewrite: Sanity-shaped types, TSanityImage, TPortableTextBlock, TLayoutBlock union, all page/document types
- `frontend/src/data/strapi-loaders.ts` - Preserved Strapi loaders (renamed from loaders.ts) for Phase 6 cleanup
- `frontend/src/data/strapi-data-api.ts` - Preserved Strapi API client (renamed from data-api.ts) for Phase 6 cleanup
- `frontend/src/components/ui/sanity-image.tsx` - New: SanityImage component using imageUrlBuilder with hotspot/crop support
- `frontend/next.config.ts` - cdn.sanity.io added as first remote pattern

## Decisions Made

- `TLayoutBlock` union moved entirely to `types/index.ts` — the old file had circular imports where types/index.ts imported from page files which imported back from types. The new design is one-directional.
- `imageUrlBuilder(client)` is initialized at module level outside the component function — avoids creating a new builder instance on every render.
- `strapi-image.tsx` left completely untouched as per D-06 — it will be removed in Phase 6.

## Deviations from Plan

None — plan executed exactly as written.

## Issues Encountered

- `frontend/` is a git sub-repo (has its own `.git`) — committed directly to the frontend sub-repo instead of using root `git add`. This matches the `sub_repos: ["backend", "frontend"]` config.

## User Setup Required

None — no external service configuration required.

## Next Phase Readiness

- Type foundation is complete — all Plans 02-04 can build against these types
- SanityImage component is ready for use in layout components (Plan 03)
- Note: TypeScript compilation will fail until Plans 02-04 wire up loaders and update consumers — this is expected per the plan's verification note
- cdn.sanity.io is in remote patterns — Sanity CDN images will render once content is migrated (Phase 5)

## Self-Check: PASSED

- frontend/src/data/strapi-loaders.ts — FOUND
- frontend/src/data/strapi-data-api.ts — FOUND
- frontend/src/types/index.ts — FOUND (TSanityImage, TPortableTextBlock, TLayoutBlock)
- frontend/src/components/ui/sanity-image.tsx — FOUND
- .planning/phases/02-data-layer-and-image-pipeline/02-01-SUMMARY.md — FOUND
- Commit 4807c7b — FOUND in frontend sub-repo
- Commit ac6488e — FOUND in frontend sub-repo

---
*Phase: 02-data-layer-and-image-pipeline*
*Completed: 2026-03-26*
