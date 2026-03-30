---
phase: 01-schema-studio-and-infrastructure
plan: 03
subsystem: schema
tags: [sanity, schema, defineType, defineField, document-types, singleton, collection, liveEdit, slug, reference]

# Dependency graph
requires:
  - phase: 01-schema-studio-and-infrastructure
    provides: "23 Sanity object type schemas: layout blocks, shared sub-objects, navigation types (Plan 02)"
provides:
  - "10 Sanity document type schemas: 7 singletons with per-page block restrictions, 3 collections with slug/enum/liveEdit"
  - "Schema barrel file exporting all 33 types as schemaTypes array for sanity.config.ts"
  - "Polymorphic navigation menu supporting menuDirectLink and menuDropdown in same array"
  - "Form submission documents (contactForm, generalInquiry) with liveEdit: true (no draft/publish)"
affects: [01-schema-studio-and-infrastructure, 02-data-layer-and-frontend-adaptation]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Document type schemas with per-page blocks array restrictions"
    - "Singleton documents for page content (homePage, contactPage, etc.)"
    - "Collection documents for repeatable content (trainingProgram) and form submissions (contactForm, generalInquiry)"
    - "liveEdit: true for form submission documents (disables draft/publish workflow)"
    - "Slug field with source: title for auto-generation"
    - "Reference array for document-to-document relationships (trainingProgramsPage -> trainingProgram)"
    - "Inline object type references for single components (globalSettings header/footer)"
    - "Enum fields as string with options.list for experience/interest dropdowns"

key-files:
  created:
    - frontend/src/sanity/schemas/documents/home-page.ts
    - frontend/src/sanity/schemas/documents/contact-page.ts
    - frontend/src/sanity/schemas/documents/gallery-page.ts
    - frontend/src/sanity/schemas/documents/training-page.ts
    - frontend/src/sanity/schemas/documents/training-programs-page.ts
    - frontend/src/sanity/schemas/documents/global-settings.ts
    - frontend/src/sanity/schemas/documents/main-menu.ts
    - frontend/src/sanity/schemas/documents/training-program.ts
    - frontend/src/sanity/schemas/documents/contact-form.ts
    - frontend/src/sanity/schemas/documents/general-inquiry.ts
    - frontend/src/sanity/schemas/index.ts
  modified: []

key-decisions:
  - "globalSettings uses inline type references (type: header, type: footer) rather than blocks array — matches Strapi repeatable: false pattern"
  - "mainMenu uses items array with polymorphic types (menuDirectLink + menuDropdown) — Sanity _type discriminator replaces Strapi __component"
  - "trainingProgramsPage uses reference array to trainingProgram documents — replaces Strapi oneToMany relation"
  - "contactForm and generalInquiry use liveEdit: true — matches Strapi draftAndPublish: false for form submissions"
  - "training-page description field uses string type (not text) — matches Strapi source schema exactly"

patterns-established:
  - "Per-page block restrictions: each singleton's blocks array only allows the specific block types that page uses"
  - "Schema barrel pattern: single index.ts exports all 33 schema types as flat array for sanity.config.ts"
  - "Form submission documents: liveEdit true, required validation on mandatory fields, enum options.list for dropdowns"
  - "Document-to-document references: array of { type: reference, to: [{ type: docType }] }"

requirements-completed: [SCHEMA-01, SCHEMA-02, SCHEMA-05]

# Metrics
duration: 6min
completed: 2026-03-25
---

# Phase 01 Plan 03: Document Type Schemas Summary

**10 Sanity document type schemas (7 singletons with per-page block restrictions, 3 collections with slug/enum/liveEdit) plus barrel file registering all 33 schema types**

## Performance

- **Duration:** 6 min
- **Started:** 2026-03-25T16:45:33Z
- **Completed:** 2026-03-25T16:51:45Z
- **Tasks:** 2
- **Files created:** 11

## Accomplishments
- All 7 singleton document types defined with per-page block type restrictions matching Strapi dynamic zone components exactly
- All 3 collection document types defined: trainingProgram (slug + blocks), contactForm (9 fields with enums and liveEdit), generalInquiry (5 fields with liveEdit)
- Navigation menu supports polymorphic items array (menuDirectLink + menuDropdown) for SCHEMA-05
- Schema barrel file registers all 33 types (10 documents + 14 layout + 6 shared + 3 navigation) with zero omissions

## Task Commits

Each task was committed atomically:

1. **Task 1: Create all 10 document type schemas** - `frontend@b05ab9a` (feat)
2. **Task 2: Create schema barrel file registering all 33 types** - `frontend@f6cdfd9` (feat)

## Files Created/Modified
- `frontend/src/sanity/schemas/documents/home-page.ts` - Homepage singleton with heroSection + featuresSection blocks
- `frontend/src/sanity/schemas/documents/contact-page.ts` - Contact page with contactInfo, schedule, contactFormLabels, generalInquiryLabels blocks
- `frontend/src/sanity/schemas/documents/gallery-page.ts` - Gallery with photoGallery + heroSection blocks
- `frontend/src/sanity/schemas/documents/training-page.ts` - Training with ctaFeaturesSection + imageWithText blocks
- `frontend/src/sanity/schemas/documents/training-programs-page.ts` - Training programs overview with reference array to trainingProgram documents
- `frontend/src/sanity/schemas/documents/global-settings.ts` - Global settings with inline header and footer objects
- `frontend/src/sanity/schemas/documents/main-menu.ts` - Navigation menu with polymorphic items (menuDirectLink + menuDropdown)
- `frontend/src/sanity/schemas/documents/training-program.ts` - Training program collection with slug, heroSection + infoSection + contactFormLabels blocks
- `frontend/src/sanity/schemas/documents/contact-form.ts` - Training contact form submission (liveEdit, experience/interest enums)
- `frontend/src/sanity/schemas/documents/general-inquiry.ts` - General inquiry submission (liveEdit, required name/email/message)
- `frontend/src/sanity/schemas/index.ts` - Barrel file exporting schemaTypes array with all 33 schema types

## Decisions Made
- globalSettings uses inline type references for header/footer (not blocks array) to match Strapi's `repeatable: false` component pattern
- mainMenu field renamed from Strapi's `MainMenuItems` to `items` for cleaner naming while preserving polymorphic behavior
- trainingProgramsPage uses Sanity reference array instead of Strapi's relation field, enabling direct document linking
- contactForm and generalInquiry set `liveEdit: true` to match Strapi's `draftAndPublish: false` — form submissions bypass draft workflow
- training-page description kept as `string` type (not `text`) matching Strapi source schema exactly

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered
None

## User Setup Required
None - no external service configuration required.

## Next Phase Readiness
- All 33 schema types (10 documents + 23 objects) are defined and registered in the barrel file
- The barrel file is ready for import by sanity.config.ts in Plan 04
- Every Strapi content type now has a Sanity counterpart with matching fields and validation

## Self-Check: PASSED

- All 11 created files verified present on disk
- Commit frontend@b05ab9a verified in git log
- Commit frontend@f6cdfd9 verified in git log

---
*Phase: 01-schema-studio-and-infrastructure*
*Completed: 2026-03-25*
