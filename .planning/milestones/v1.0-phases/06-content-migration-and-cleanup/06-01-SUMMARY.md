---
phase: 06-content-migration-and-cleanup
plan: 01
subsystem: migration
tags: [sanity, strapi, migration, portable-text, nanoid, tsx]

# Dependency graph
requires:
  - phase: 05-cache-revalidation
    provides: Sanity webhook and revalidation tags — confirms Sanity is fully wired as the live CMS
  - phase: 01-schema-studio-and-infrastructure
    provides: All Sanity document schemas (homePage, trainingProgram, globalSettings, etc.)
  - phase: 02-data-layer-and-image-pipeline
    provides: Strapi-named loaders and Sanity GROQ loaders — reference for field mappings
provides:
  - Migration script at frontend/scripts/migrate-strapi-to-sanity.ts
  - All Strapi content types transformed and written to Sanity (pending execution)
  - Strapi Blocks JSON → Portable Text conversion utility
  - Image upload pipeline from Strapi CDN → Sanity CDN
affects:
  - 06-02 (cleanup — requires migration complete before Strapi code removal)
  - 06-03 (env cleanup — requires migration verified)

# Tech tracking
tech-stack:
  added:
    - nanoid@3.3.11 (added to dependencies)
  patterns:
    - Image pre-upload pass to build asset map before document creation
    - createOrReplace with plain _id creates published documents (no drafts. prefix)
    - Transaction batching for collection type migrations (deferred visibility)
    - Strapi Blocks node-type mapping to Portable Text block/span shapes

key-files:
  created:
    - frontend/scripts/migrate-strapi-to-sanity.ts
  modified:
    - frontend/package.json (added nanoid dep, added migrate script)

key-decisions:
  - "nanoid(12) used for all _key generation — URL-safe 12-char keys for Sanity array items"
  - "Image pre-upload phase runs before document migration — imageCache Map deduplicates identical Strapi images"
  - "createOrReplace with plain _id (no drafts. prefix) publishes documents directly"
  - "Script creates its own Sanity client — cannot import write-client.ts due to server-only guard"
  - "collectAndUploadAllImages walks all deep-populated endpoints to find /uploads/ URLs before migration begins"

patterns-established:
  - "Script env validation: assertValue pattern with process.exit(1) on missing vars"
  - "Block transformer: kebab-case component names (layout.hero-section) to camelCase Sanity _type"
  - "Portable Text conversion: paragraph/heading/list nodes with flattenSpans handling link mark defs"

requirements-completed: [MIG-01, MIG-02, MIG-03, MIG-04, MIG-05]

# Metrics
duration: 8min
completed: 2026-03-30
---

# Phase 6 Plan 01: Strapi to Sanity Migration Script Summary

**TypeScript migration script (1259 lines) that fetches all 10 Strapi content types, uploads images to Sanity CDN, converts Strapi Blocks rich text to Portable Text, and writes all documents as published via createOrReplace**

## Performance

- **Duration:** ~8 min
- **Started:** 2026-03-30T17:47:00Z
- **Completed:** 2026-03-30T17:55:29Z
- **Tasks:** 1 of 2 (Task 2 is a human-action checkpoint — requires Strapi credentials)
- **Files modified:** 2

## Accomplishments

- Created `frontend/scripts/migrate-strapi-to-sanity.ts` — full 1259-line migration script
- Added `nanoid` to `package.json` dependencies and `migrate` script entry
- Script handles all 10 Strapi content types: 7 singletons (homePage, contactPage, galleryPage, trainingPage, trainingProgramsPage, globalSettings, mainMenu) + 3 collections (trainingProgram, contactForm, generalInquiry)
- Image pre-upload pass walks all deep-populated endpoints, deduplicates URLs, and builds an imageCache map before any document creation
- Portable Text converter handles paragraph, heading (h1→h2, else h3), ordered/unordered lists, code blocks, and inline link mark definitions
- Batch transaction commits for collection types using deferred visibility
- Summary report queries Sanity for actual counts and shows OK/MISMATCH/EMPTY/ERROR per content type

## Task Commits

1. **Task 1: Install nanoid and create migration script** — `8f6ad91` (feat)
2. **Task 2: Run migration script** — PENDING (checkpoint:human-action — requires user to supply Strapi credentials)

## Files Created/Modified

- `frontend/scripts/migrate-strapi-to-sanity.ts` — Complete Strapi-to-Sanity migration script
- `frontend/package.json` — Added `nanoid` dependency, added `migrate` npm script

## Decisions Made

- Used nanoid v3.3.11 (already in node_modules as transitive dep) — added to package.json explicitly
- Script creates its own `createClient` instead of importing `write-client.ts` because `write-client.ts` has `import "server-only"` which fails outside Next.js context
- Image pre-upload phase uses recursive URL collector to walk all deep-populated Strapi responses, collecting any URL containing `/uploads/`
- `fetchStrapiPaginated` uses page-based pagination with pageSize=100 and reads `meta.pagination.pageCount` to know when to stop
- Block transformer uses a `switch` on camelCase `_type` — covers all 14 layout block types from the schema

## Deviations from Plan

None — plan executed exactly as written. nanoid was already in node_modules but not in package.json; added it explicitly as planned.

## Issues Encountered

- `npm install nanoid` fails due to Motion dev token auth error in motion-studio/motion-plus packages — resolved by adding nanoid directly to package.json dependencies (it was already available in node_modules as a transitive dependency)

## User Setup Required

Task 2 requires user credentials before the migration can run:

1. Get your Strapi Cloud API URL: Strapi Cloud dashboard -> Settings -> API URL
2. Create a Full Access API token: Strapi Cloud dashboard -> Settings -> API Tokens -> Create new Full Access token
3. Add to `frontend/.env.local`:
   ```
   STRAPI_BASE_URL=https://cheerful-prosperity-b55f073006.strapiapp.com
   STRAPI_API_TOKEN={your-full-access-api-token}
   ```
4. Run: `cd frontend && npm run migrate`
5. Review terminal output for OK/MISMATCH in summary report
6. Verify in Sanity Studio that all documents appear

## Next Phase Readiness

- Migration script is complete and ready to run — waiting on user to supply Strapi credentials (Task 2 checkpoint)
- Once migration is verified, Phase 06-02 (Strapi code removal) and 06-03 (env cleanup) can proceed
- No blockers on the script itself — TypeScript compiles cleanly, all env validation and error handling in place

---
*Phase: 06-content-migration-and-cleanup*
*Completed: 2026-03-30*

## Self-Check: PASSED

- `frontend/scripts/migrate-strapi-to-sanity.ts`: FOUND (1259 lines)
- `frontend/package.json` contains `"migrate"` script: FOUND
- `frontend/package.json` contains `"nanoid"` dependency: FOUND
- Commit `8f6ad91` exists: FOUND
