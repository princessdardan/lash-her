---
phase: 02-data-layer-and-image-pipeline
plan: "02"
subsystem: data-layer
tags: [groq, loaders, sanity, block-renderer, data-api]
dependency_graph:
  requires: ["02-01"]
  provides: ["loaders.ts with GROQ loaders", "block-renderer with Sanity _type dispatch"]
  affects: ["frontend/src/data/loaders.ts", "frontend/src/components/custom/layouts/block-renderer.tsx"]
tech_stack:
  added: ["groq template tag from next-sanity"]
  patterns: ["client.fetch<T>(groq`...`, params)", "singleton *[_type][0] queries", "collection *[_type] queries"]
key_files:
  created:
    - frontend/src/data/loaders.ts
  modified:
    - frontend/src/components/custom/layouts/block-renderer.tsx
decisions:
  - "getAllTrainingProgramSlugs added as 10th loader for generateStaticParams support"
  - "TLayoutBlock re-exported from block-renderer for backward compatibility"
  - "Prop type imports removed from block-renderer (IHeroSectionProps etc) — replaced by types/index.ts"
metrics:
  duration: "~3min"
  completed: "2026-03-26"
  tasks: 2
  files: 2
---

# Phase 02 Plan 02: GROQ Loaders and Block Renderer Update Summary

GROQ-based data loaders with 10 fetch functions and Sanity _type camelCase block registry replacing all Strapi-era `__component` and `layout.` prefixes.

## Tasks Completed

| Task | Name | Commit | Files |
|------|------|--------|-------|
| 1 | Create GROQ-based loaders.ts with all data fetch functions | frontend@ed1e7c8 | frontend/src/data/loaders.ts (created) |
| 2 | Update block-renderer.tsx for Sanity _type dispatch | frontend@b0d542f | frontend/src/components/custom/layouts/block-renderer.tsx (modified) |

## What Was Built

### Task 1: GROQ Data Loaders (`frontend/src/data/loaders.ts`)

New file with 10 GROQ-based loader functions:

- `getHomePageData()` — singleton fetch for homePage document
- `getContactPageData()` — singleton fetch for contactPage document
- `getGalleryPageData()` — singleton fetch for galleryPage document
- `getTrainingsPageData()` — singleton fetch for trainingPage document
- `getGlobalData()` — singleton fetch for globalSettings with header/footer
- `getMainMenuData()` — singleton fetch for mainMenu with polymorphic items
- `getMetaData()` — lightweight singleton fetch for title/description only
- `getTrainingProgramBySlug(slug)` — collection fetch filtered by `slug.current == $slug`
- `getAllTrainingPrograms()` — collection fetch for all programs with full blocks
- `getAllTrainingProgramSlugs()` — lightweight collection fetch for generateStaticParams

All functions use `client.fetch<T>(groq`...`, params)` directly. No Strapi patterns (no `qs`, no `api`, no `TStrapiResponse`, no response envelope). Singletons return `T | null`, collections return `T[]`. Exports via `export const loaders = { ... }` object.

### Task 2: Block Renderer (`frontend/src/components/custom/layouts/block-renderer.tsx`)

Updated `COMPONENT_REGISTRY` keys from Strapi dotted format to Sanity camelCase:

| Old Key | New Key |
|---------|---------|
| `layout.hero-section` | `heroSection` |
| `layout.features-section` | `featuresSection` |
| `layout.cta-features-section` | `ctaFeaturesSection` |
| `layout.image-with-text` | `imageWithText` |
| `layout.info-section` | `infoSection` |
| `layout.photo-gallery` | `photoGallery` |
| `layout.schedule` | `schedule` |
| `layout.contact-info` | `contactInfo` |
| `layout.contact-form` | `contactFormLabels` |

`NON_RENDERABLE_COMPONENTS`: `layout.general-inquiry-labels` → `generalInquiryLabels`
`SKIP_ANIMATION`: `layout.hero-section` → `heroSection`

`BaseBlock` interface changed from `{ __component: string; id?: number; documentId?: string }` to `{ _type: string; _key?: string }`.

`getBlockKey` now uses `block._key` instead of `block.documentId`/`block.id`.

`TLayoutBlock` local union removed — now imported from `@/types` and re-exported for backward compatibility. Prop type imports (IHeroSectionProps etc.) removed since they were only needed for the local union.

All rendering logic, error boundaries, animation wrappers, Suspense support, and component API preserved unchanged.

## Deviations from Plan

None — plan executed exactly as written.

## Known Stubs

None. The loaders.ts GROQ projections are complete. Block renderer wires to existing React components (which still use Strapi-shaped props — those are fixed in Plan 03).

## Self-Check: PASSED

- `frontend/src/data/loaders.ts` — FOUND
- `frontend/src/components/custom/layouts/block-renderer.tsx` — FOUND
- Commit `ed1e7c8` — verified via commit-to-subrepo output
- Commit `b0d542f` — verified via commit-to-subrepo output
- `grep -c "client.fetch" loaders.ts` = 10 (>=10 required)
- `grep -c "__component" block-renderer.tsx` = 0 (required)
- `grep -c "heroSection" block-renderer.tsx` = 2 (>=1 required)
- `grep -c "layout.hero-section" block-renderer.tsx` = 0 (required)
