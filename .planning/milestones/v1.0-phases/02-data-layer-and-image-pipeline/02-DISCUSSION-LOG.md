# Phase 2: Data Layer and Image Pipeline - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-03-25
**Phase:** 02-data-layer-and-image-pipeline
**Areas discussed:** GROQ loader organization, Image component design, Block renderer transition, Type definitions strategy

---

## GROQ Loader Organization

| Option | Description | Selected |
|--------|-------------|----------|
| Single file | Keep existing loaders.ts pattern — one file exporting a loaders object with all functions. GROQ queries inline. | ✓ |
| Single file, extracted queries | One loaders.ts file, but GROQ query strings extracted as named constants at the top or separate queries.ts. | |
| Split per page | Separate files per page (home-loader.ts, contact-loader.ts, etc.) | |

**User's choice:** Single file
**Notes:** Mirrors current architecture, minimal structural change.

| Option | Description | Selected |
|--------|-------------|----------|
| Import client directly | Each loader imports Sanity read client and calls client.fetch<T>(query, params). | ✓ |
| Wrapper function | Create a sanityFetch() helper wrapping client.fetch with error handling. | |

**User's choice:** Import client directly
**Notes:** Matches next-sanity conventions.

| Option | Description | Selected |
|--------|-------------|----------|
| Keep until Phase 6 | Rename Strapi files (strapi-loaders.ts, strapi-data-api.ts). New Sanity loaders in fresh file. | ✓ |
| Delete immediately | Remove Strapi data layer now. All pages switch to Sanity immediately. | |

**User's choice:** Keep until Phase 6
**Notes:** Live Strapi site keeps working during migration.

---

## Image Component Design

| Option | Description | Selected |
|--------|-------------|----------|
| Sanity-native shape | Accept raw Sanity image object directly. Use @sanity/image-url builder internally. | ✓ |
| Same interface as StrapiImage | Accept src/alt/width/height like today. Extract URL at caller site. | |

**User's choice:** Sanity-native shape
**Notes:** Components pass raw Sanity image field — no manual URL extraction needed.

| Option | Description | Selected |
|--------|-------------|----------|
| Yes, full support | Use @sanity/image-url for crop-aware URLs. Editors set focal points in Studio. | ✓ |
| Basic only | Just build image URLs without hotspot/crop. | |

**User's choice:** Full hotspot/crop support
**Notes:** None.

| Option | Description | Selected |
|--------|-------------|----------|
| Keep until Phase 6 | Old strapi-image.tsx stays. New sanity-image.tsx created alongside. | ✓ |
| Delete now | Remove strapi-image.tsx immediately. | |

**User's choice:** Keep until Phase 6
**Notes:** None.

---

## Block Renderer Transition

| Option | Description | Selected |
|--------|-------------|----------|
| Big bang switch | Update COMPONENT_REGISTRY keys all at once. Change __component to _type. | ✓ |
| Dual registry | Support both __component and _type keys during migration. | |

**User's choice:** Big bang switch
**Notes:** Since all pages switch to Sanity loaders in this phase, no period where both formats coexist.

| Option | Description | Selected |
|--------|-------------|----------|
| Yes, update all | Update NON_RENDERABLE_COMPONENTS and SKIP_ANIMATION to Sanity _type names. | ✓ |
| Defer to later | Only update COMPONENT_REGISTRY now. | |

**User's choice:** Update all auxiliary sets
**Notes:** Keeps everything consistent.

---

## Type Definitions Strategy

| Option | Description | Selected |
|--------|-------------|----------|
| Single barrel file | Keep types/index.ts. Replace Strapi types with Sanity types. Remove TStrapiResponse. | ✓ |
| Split by domain | Separate files: documents.ts, objects.ts, blocks.ts, forms.ts with barrel re-exports. | |
| Co-locate with schemas | Types live next to Sanity schemas in frontend/src/sanity/types/. | |

**User's choice:** Single barrel file
**Notes:** Minimal structural change. GROQ returns plain objects — no response wrapper needed.

| Option | Description | Selected |
|--------|-------------|----------|
| Remove Strapi types | Delete TStrapiResponse, Strapi TImage, BlocksContent, etc. Clean break. | ✓ |
| Keep until Phase 6 | Rename with Strapi prefix and keep alongside. | |

**User's choice:** Remove Strapi types
**Notes:** Nothing references them after loaders switch to GROQ.

| Option | Description | Selected |
|--------|-------------|----------|
| Move to types/index.ts | Define all block unions centrally. Pages import from @/types. | ✓ |
| Keep in page files | Each page.tsx continues to define its own block union type. | |

**User's choice:** Move to types/index.ts
**Notes:** Eliminates circular import pattern where types/index.ts imports from page files.

---

## Claude's Discretion

- GROQ query projections, image builder config, Sanity type field details, getBlockKey updates, utility function handling

## Deferred Ideas

None — discussion stayed within phase scope.
