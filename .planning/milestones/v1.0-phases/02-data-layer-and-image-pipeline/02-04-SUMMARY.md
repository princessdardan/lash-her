---
phase: 02-data-layer-and-image-pipeline
plan: 04
subsystem: frontend/pages
tags: [sanity, groq, pages, loaders, migration]
dependency_graph:
  requires: [02-01, 02-02, 02-03]
  provides: [all-pages-on-sanity, groq-static-params]
  affects: [frontend/src/app/(site), frontend/src/components/custom/contact-content.tsx]
tech_stack:
  added: []
  patterns: [groq-loaders, null-check-notFound, Sanity _type discrimination]
key_files:
  created: []
  modified:
    - frontend/src/app/(site)/layout.tsx
    - frontend/src/app/(site)/page.tsx
    - frontend/src/app/(site)/contact/page.tsx
    - frontend/src/app/(site)/gallery/page.tsx
    - frontend/src/app/(site)/training/page.tsx
    - frontend/src/app/(site)/training-programs/[slug]/page.tsx
    - frontend/src/components/custom/contact-content.tsx
    - frontend/src/app/layout.tsx
decisions:
  - All pages use direct null-check pattern (`if (!data) notFound()`) — no validateApiResponse wrapper needed with GROQ loaders that return T | null
  - root app/layout.tsx metadata also had .data wrapper — fixed as Rule 1 (bug auto-fix)
metrics:
  duration: 218s
  completed: 2026-03-26
---

# Phase 02 Plan 04: Wire Pages to Sanity GROQ Loaders Summary

**One-liner:** All 5 content pages and site layout wired to Sanity GROQ loaders, replacing validateApiResponse/TStrapiResponse with direct null guards and _type block discrimination.

## What Was Built

- Site layout (`(site)/layout.tsx`) now uses `loaders.getGlobalData()` and `loaders.getMainMenuData()` directly inside `unstable_cache`, no `validateApiResponse`. Menu items accessed via `.items` (Sanity field name).
- Homepage (`page.tsx`) fetches all data in parallel, null-guards with `notFound()`, passes plain data to `BlockRenderer` and `ContactContent`.
- Contact page finds blocks by `_type === "schedule"`, `"contactInfo"`, `"generalInquiryLabels"` instead of `__component`.
- Gallery page: simple null-guard + `BlockRenderer`.
- Training page: `_type === "heroSection"` replaces `__component === "layout.hero-section"`.
- Training programs `[slug]/page.tsx`: `generateStaticParams` now calls `loaders.getAllTrainingProgramSlugs()` GROQ query, replacing hardcoded slug array.
- `contact-content.tsx`: updated `pageData` interface drops `id`/`documentId`, block finds use `_type`.

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 1 - Bug] Fixed root layout.tsx metadata using .data wrapper**
- **Found during:** Task 2 TypeScript check
- **Issue:** `src/app/layout.tsx` still had `metadata?.data?.title` (Strapi pattern) causing TS2339 errors
- **Fix:** Updated to `metadata?.title` and `metadata?.description` to match GROQ loader return type
- **Files modified:** `frontend/src/app/layout.tsx`
- **Commit:** 468c31a

### Pre-existing Issues (Out of Scope, Deferred)

The following TypeScript errors existed before this plan and are not caused by changes here:
- `src/components/custom/layouts/cta-section-video.tsx` — `TVideo` not in types
- `src/components/custom/layouts/cta-section-image.tsx` — `TImage` not in types
- `src/components/custom/cta-feature.tsx` — `BlocksContent` not in types
- `src/components/ui/block-renderer.tsx` — `BlocksContent`, `InlineNode` not in types
- `src/components/ui/mobile-navigation.tsx` — still uses `__component` and Strapi field names
- `src/data/strapi-data-api.ts` / `strapi-loaders.ts` — reference removed Strapi types
- `src/app/(site)/global-errors.tsx` — `id` not in `TLink`

## Known Stubs

None — all pages are wired to live Sanity loaders returning real data.

## Self-Check: PASSED
