---
phase: 01-schema-studio-and-infrastructure
plan: 02
subsystem: schema
tags: [sanity, schema, defineType, defineField, portable-text, object-types]

# Dependency graph
requires:
  - phase: 01-schema-studio-and-infrastructure
    provides: "Sanity packages installed, schema directory structure created (Plan 01)"
provides:
  - "23 Sanity object type schemas: 6 reusable sub-objects, 3 navigation types, 14 layout blocks"
  - "Portable Text configuration for rich text fields (perks, info, features)"
  - "Image fields with hotspot and alt text pattern"
  - "Enum fields matching Strapi source values"
affects: [01-schema-studio-and-infrastructure, 02-data-layer-and-frontend-adaptation]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "defineType/defineField pattern for all Sanity object schemas"
    - "Portable Text block config with styles (normal, h2, h3), lists (bullet, number), marks (strong, em, link)"
    - "Image with hotspot: true and nested alt text sub-field"
    - "Enum fields as string type with options.list array"
    - "Single object reference via type name directly (not array)"

key-files:
  created:
    - frontend/src/sanity/schemas/objects/shared/link.ts
    - frontend/src/sanity/schemas/objects/shared/menu-link.ts
    - frontend/src/sanity/schemas/objects/shared/feature.ts
    - frontend/src/sanity/schemas/objects/shared/cta-feature.ts
    - frontend/src/sanity/schemas/objects/shared/contact.ts
    - frontend/src/sanity/schemas/objects/shared/hours.ts
    - frontend/src/sanity/schemas/objects/shared/menu-direct-link.ts
    - frontend/src/sanity/schemas/objects/shared/menu-dropdown.ts
    - frontend/src/sanity/schemas/objects/shared/menu-dropdown-section.ts
    - frontend/src/sanity/schemas/objects/layout/hero-section.ts
    - frontend/src/sanity/schemas/objects/layout/features-section.ts
    - frontend/src/sanity/schemas/objects/layout/cta-features-section.ts
    - frontend/src/sanity/schemas/objects/layout/cta-section-image.ts
    - frontend/src/sanity/schemas/objects/layout/cta-section-video.ts
    - frontend/src/sanity/schemas/objects/layout/image-with-text.ts
    - frontend/src/sanity/schemas/objects/layout/info-section.ts
    - frontend/src/sanity/schemas/objects/layout/photo-gallery.ts
    - frontend/src/sanity/schemas/objects/layout/schedule.ts
    - frontend/src/sanity/schemas/objects/layout/contact-info.ts
    - frontend/src/sanity/schemas/objects/layout/contact-form-labels.ts
    - frontend/src/sanity/schemas/objects/layout/general-inquiry-labels.ts
    - frontend/src/sanity/schemas/objects/layout/header.ts
    - frontend/src/sanity/schemas/objects/layout/footer.ts
  modified: []

key-decisions:
  - "menuDropdownSection inlines the skipped Strapi section collection type per D-11"
  - "contactFormLabels and generalInquiryLabels use renamed type names per D-12"
  - "Rich text fields (perks, info, ctaFeature.features) use Portable Text block type per D-13"
  - "photoGallery.images is images-only (no video) per research recommendation"

patterns-established:
  - "Portable Text config: styles [normal, h2, h3], lists [bullet, number], marks [strong, em], annotations [link with href + blank]"
  - "Image field pattern: type image, hotspot true, nested alt string field"
  - "Enum pattern: type string with options.list array of {title, value} pairs"
  - "Single component reference: use type name directly as field type (not wrapped in array)"
  - "Array component reference: type array with of [{type: 'typeName'}]"

requirements-completed: [SCHEMA-03, SCHEMA-04, SCHEMA-05]

# Metrics
duration: 4min
completed: 2026-03-25
---

# Phase 01 Plan 02: Object Type Schemas Summary

**23 Sanity object type schemas mirroring all Strapi components field-for-field: 6 reusable sub-objects (link, menuLink, feature, ctaFeature, contact, hours), 3 navigation types (menuDirectLink, menuDropdown, menuDropdownSection), and 14 layout blocks with Portable Text for rich text fields**

## Performance

- **Duration:** 4 min
- **Started:** 2026-03-25T16:07:27Z
- **Completed:** 2026-03-25T16:11:28Z
- **Tasks:** 2
- **Files created:** 23

## Accomplishments
- All 6 reusable sub-object types defined with fields matching Strapi source schemas exactly
- All 3 navigation-specific types support polymorphic menu items (menuDirectLink for direct links, menuDropdown with sections of menuLink items)
- All 14 layout block types defined with matching fields, types, and enumerations
- Rich text fields (imageWithText.perks, infoSection.info, ctaFeature.features) use Portable Text block type with consistent configuration
- Image fields use hotspot: true with nested alt text sub-field throughout
- Enum values match Strapi exactly: EYE_ICON, SPARKLES_ICON, STAR_ICON, VIDEO_ICON, USER_ICON, USERS_ICON, AWARD_ICON, HORIZONTAL_IMAGE_LEFT, HORIZONTAL_IMAGE_RIGHT, VERTICAL

## Task Commits

Each task was committed atomically:

1. **Task 1: Create reusable sub-object schemas and navigation types** - `frontend@1868dc0` (feat)
2. **Task 2: Create layout block object schemas** - `frontend@2c4bf43` (feat)

## Files Created/Modified
- `frontend/src/sanity/schemas/objects/shared/link.ts` - Reusable link object (href, label, isExternal)
- `frontend/src/sanity/schemas/objects/shared/menu-link.ts` - Menu link with name, url, description, icon image
- `frontend/src/sanity/schemas/objects/shared/feature.ts` - Feature with heading, subHeading, icon enum
- `frontend/src/sanity/schemas/objects/shared/cta-feature.ts` - CTA feature with Portable Text features, link, icon enum, mostPopular
- `frontend/src/sanity/schemas/objects/shared/contact.ts` - Contact info (phone, email, location)
- `frontend/src/sanity/schemas/objects/shared/hours.ts` - Business hours (days, times)
- `frontend/src/sanity/schemas/objects/shared/menu-direct-link.ts` - Direct navigation link (title, url)
- `frontend/src/sanity/schemas/objects/shared/menu-dropdown.ts` - Dropdown with sections of menuDropdownSection
- `frontend/src/sanity/schemas/objects/shared/menu-dropdown-section.ts` - Dropdown section with heading and menuLink array
- `frontend/src/sanity/schemas/objects/layout/hero-section.ts` - Hero section with image, heading, links, onHomepage
- `frontend/src/sanity/schemas/objects/layout/features-section.ts` - Features section with feature array
- `frontend/src/sanity/schemas/objects/layout/cta-features-section.ts` - CTA features with ctaFeature array
- `frontend/src/sanity/schemas/objects/layout/cta-section-image.ts` - CTA section with image and links
- `frontend/src/sanity/schemas/objects/layout/cta-section-video.ts` - CTA section with video file and links
- `frontend/src/sanity/schemas/objects/layout/image-with-text.ts` - Image with text, Portable Text perks, orientation enum
- `frontend/src/sanity/schemas/objects/layout/info-section.ts` - Info section with Portable Text info field
- `frontend/src/sanity/schemas/objects/layout/photo-gallery.ts` - Photo gallery with image array (hotspot + alt)
- `frontend/src/sanity/schemas/objects/layout/schedule.ts` - Schedule with hours array
- `frontend/src/sanity/schemas/objects/layout/contact-info.ts` - Contact info with contact array
- `frontend/src/sanity/schemas/objects/layout/contact-form-labels.ts` - Training form UI labels (D-12 renamed)
- `frontend/src/sanity/schemas/objects/layout/general-inquiry-labels.ts` - General inquiry form UI labels (D-12 renamed)
- `frontend/src/sanity/schemas/objects/layout/header.ts` - Header with single logoText link and ctaButton array
- `frontend/src/sanity/schemas/objects/layout/footer.ts` - Footer with single logoText link, text, socialLink array

## Decisions Made
- menuDropdownSection inlines what was Strapi's `section` collection type (heading + links array of menuLink), per D-11 decision to skip that collection type
- contactFormLabels and generalInquiryLabels use camelCase renamed type names per D-12 to avoid collision with form submission document types
- All rich text fields (Strapi `blocks` type) mapped to Portable Text with consistent block configuration per D-13
- photoGallery uses images-only (no video support in the array) per research recommendation, matching primary use case

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered
None

## User Setup Required
None - no external service configuration required.

## Next Phase Readiness
- All 23 object type schemas are ready to be referenced by document schemas in Plan 03
- The `blocks` arrays in document types can now reference these layout block types
- Navigation menu documents can use menuDirectLink and menuDropdown types

## Self-Check: PASSED

- All 23 schema files verified present on disk
- Commit frontend@1868dc0 verified in git log
- Commit frontend@2c4bf43 verified in git log

---
*Phase: 01-schema-studio-and-infrastructure*
*Completed: 2026-03-25*
