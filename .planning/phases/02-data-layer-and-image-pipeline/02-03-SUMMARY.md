---
phase: 02-data-layer-and-image-pipeline
plan: 03
subsystem: ui
tags: [sanity, typescript, react, next.js, portable-text]

requires:
  - phase: 02-01
    provides: TSanityImage, TPortableTextBlock, and all Sanity block type interfaces in types/index.ts
  - phase: 02-02
    provides: Sanity-based GROQ loaders returning Sanity-shaped data

provides:
  - All 11 layout/collection components accept Sanity-shaped props (_type/_key, TSanityImage, TPortableTextBlock[])
  - SanityImage replaces StrapiImage in hero-section, gallery, image-with-text
  - Gallery uses images (plural) field matching Sanity photoGallery schema
  - Rich text fields (perks, info, cta features) typed as TPortableTextBlock[] with plain-text stub
  - Main menu discriminates on _type (menuDirectLink/menuDropdown) instead of __component
  - All components export backward-compatible type aliases for existing importers

affects:
  - 02-04 (page-level wiring — pages now pass Sanity loader data to these components)
  - 03 (portable text renderer — replaces stubs in image-with-text, info-section, cta-features-section)

tech-stack:
  added: []
  patterns:
    - "Portable Text stub pattern: plain text extraction via block.children.map(child => child.text).join('') pending Phase 3 renderer"
    - "Backward-compat type alias export: export type { TSanityType as ILegacyProps } from '@/types'"

key-files:
  created: []
  modified:
    - frontend/src/components/custom/layouts/hero-section.tsx
    - frontend/src/components/custom/layouts/gallery.tsx
    - frontend/src/components/custom/layouts/image-with-text.tsx
    - frontend/src/components/custom/layouts/features-section.tsx
    - frontend/src/components/custom/layouts/cta-features-section.tsx
    - frontend/src/components/custom/layouts/info-section.tsx
    - frontend/src/components/custom/layouts/schedule.tsx
    - frontend/src/components/custom/layouts/contact-info.tsx
    - frontend/src/components/custom/collection/contact-components.tsx
    - frontend/src/components/custom/collection/general-inquiry.tsx
    - frontend/src/app/main-menu.tsx

key-decisions:
  - "Portable Text stub uses plain text extraction (block.children.map(child => child.text)) — renders readable text until Phase 3 PortableText renderer is wired in"
  - "IGeneralInquiryLayoutProps and IContactPageLayoutProps drop id/documentId fields — Sanity document projections don't include these Strapi-specific fields"
  - "Out-of-scope files cta-section-video.tsx and cta-section-image.tsx still use __component — not in plan scope, logged as deferred"

patterns-established:
  - "Backward-compat type alias: components export ILegacyName as type alias to the canonical Sanity type"
  - "Rich text stub: render plain text from TPortableTextBlock[] until Phase 3 PortableText renderer"
  - "Key assignment: use item._key || index for React keys in array renders"

requirements-completed:
  - DATA-05
  - IMG-02

duration: 4min
completed: 2026-03-26
---

# Phase 02 Plan 03: Component Prop Migration Summary

**All 11 layout and collection components migrated to Sanity-shaped props: SanityImage replaces StrapiImage, _type/_key replaces __component/id, TPortableTextBlock[] stubs replace BlocksContent, and the menu discriminates on _type**

## Performance

- **Duration:** 4 min
- **Started:** 2026-03-26T01:53:43Z
- **Completed:** 2026-03-26T01:57:43Z
- **Tasks:** 2
- **Files modified:** 11

## Accomplishments
- Three image-bearing components (hero-section, gallery, image-with-text) now use SanityImage and TSanityImage props; gallery field name corrected from `image` to `images`
- Five non-image layout components and two collection form-label components drop all Strapi-specific fields (id, documentId, __component) and use canonical Sanity block types
- Main menu rewritten to dispatch on `_type` (menuDirectLink/menuDropdown) using TMenuDirectLink/TMenuDropdown instead of the Strapi `__component` string

## Task Commits

1. **Task 1: Update image-bearing layout components** - `frontend@5a3cc67` (feat)
2. **Task 2: Update non-image layout components, form labels, and main menu** - `frontend@c9d63d0` (feat)

## Files Created/Modified
- `frontend/src/components/custom/layouts/hero-section.tsx` - SanityImage, THeroSection props, _key keys on link.map
- `frontend/src/components/custom/layouts/gallery.tsx` - TSanityImage, TPhotoGallery props, images (plural) field, SanityImage
- `frontend/src/components/custom/layouts/image-with-text.tsx` - SanityImage, TImageWithText props, TPortableTextBlock[] stub for perks
- `frontend/src/components/custom/layouts/features-section.tsx` - TFeaturesSection props, _key keys
- `frontend/src/components/custom/layouts/cta-features-section.tsx` - TCtaFeaturesSection props, Portable Text stub for features
- `frontend/src/components/custom/layouts/info-section.tsx` - TInfoSection props, Portable Text stub for info
- `frontend/src/components/custom/layouts/schedule.tsx` - TSchedule/THours props, _key keys
- `frontend/src/components/custom/layouts/contact-info.tsx` - TContactInfo/TContact props, _key keys
- `frontend/src/components/custom/collection/contact-components.tsx` - TContactFormLabels, removed id/documentId from layout props
- `frontend/src/components/custom/collection/general-inquiry.tsx` - TGeneralInquiryLabels, TSchedule/TContactInfo in layout props
- `frontend/src/app/main-menu.tsx` - TMainMenuItem, _type dispatch (menuDirectLink/menuDropdown), _key keys

## Decisions Made
- Portable Text stub pattern established: plain text extraction from `block.children` array as temporary rendering until Phase 3 wires in the `@portabletext/react` renderer
- `IGeneralInquiryLayoutProps` and `IContactPageLayoutProps` remove `id: number` and `documentId: string` — Sanity document projections don't surface these Strapi-only fields
- All backward-compat type aliases exported so existing page importers continue to compile without changes

## Deviations from Plan

None - plan executed exactly as written.

## Known Stubs

Three components use a Portable Text plain-text stub pending the Phase 3 renderer:

1. `frontend/src/components/custom/layouts/image-with-text.tsx` — `perks` field renders as plain `<p>` tags from `block.children` text. Intentional: Phase 3 wires `@portabletext/react`.
2. `frontend/src/components/custom/layouts/info-section.tsx` — `info` field renders as plain `<p>` tags. Same reason.
3. `frontend/src/components/custom/layouts/cta-features-section.tsx` — `item.features` field renders as plain `<p>` tags. Same reason.

These stubs make content visible but without heading styles, lists, or inline marks. The plan goal (components accept Sanity-shaped data) is achieved; full rich text rendering is Phase 3's goal.

## Issues Encountered

Sub-repo commit routing: initial commit accidentally went to the root repo. Resolved by committing in `frontend/` sub-repo directly and restoring root repo state. Both task commits are correctly in the `frontend` sub-repo.

## Next Phase Readiness
- All components ready to accept data from Sanity GROQ loaders built in Plan 02
- Plan 04 can wire page-level data passing (loaders → pages → components)
- Two layout files (cta-section-video.tsx, cta-section-image.tsx) still use `__component` — not in scope for this plan, handled in Plan 04 or as separate deferred item

---
*Phase: 02-data-layer-and-image-pipeline*
*Completed: 2026-03-26*
