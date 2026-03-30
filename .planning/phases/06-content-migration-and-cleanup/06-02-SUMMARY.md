---
phase: 06-content-migration-and-cleanup
plan: 02
subsystem: infra
tags: [strapi, sanity, cleanup, next-config, utils, package-json]

# Dependency graph
requires:
  - phase: 06-01
    provides: Migration script complete — Strapi-to-Sanity data migration done
  - phase: 02-data-layer-and-image-pipeline
    provides: Strapi loaders/API client renamed (not deleted) in Phase 2 — now safe to remove
provides:
  - Zero Strapi references remaining in frontend/src/
  - backend/ directory deleted
  - utils.ts exports only cn() utility
  - next.config.ts has only cdn.sanity.io remote pattern
  - qs, @types/qs, @vercel/blob packages removed from package.json
affects:
  - 06-03 (env cleanup — backend gone, Strapi env vars now the last remnant)

# Tech tracking
tech-stack:
  added: []
  patterns:
    - Direct package.json edit for package removal when npm uninstall fails due to token-gated deps

key-files:
  created: []
  modified:
    - frontend/src/lib/utils.ts (reduced to cn() only)
    - frontend/next.config.ts (Sanity CDN only remote pattern)
    - frontend/package.json (removed qs, @types/qs, @vercel/blob, 4 Strapi scripts)
  deleted:
    - frontend/src/data/strapi-loaders.ts
    - frontend/src/data/strapi-data-api.ts
    - frontend/src/components/ui/strapi-image.tsx
    - frontend/src/components/ui/strapi-video.tsx
    - frontend/src/lib/error-handler.ts
    - frontend/scripts/optimize-avif.ts
    - frontend/scripts/upload-landing-images.ts
    - frontend/scripts/apply-optimization.ts
    - backend/ (entire directory)

key-decisions:
  - "npm uninstall fails for qs/@types/qs/@vercel/blob due to motion-plus/motion-studio token-gated URLs — edited package.json directly (packages already absent from active code)"
  - "Build failure (styled-components not found in @sanity/ui node_modules) is pre-existing, not caused by this cleanup — confirmed via git stash test"

patterns-established:
  - "Pre-existing build failures confirmed via git stash before/after comparison"

requirements-completed: [CLEAN-01, CLEAN-02, CLEAN-03, CLEAN-04, CLEAN-05]

# Metrics
duration: 4min
completed: 2026-03-30
---

# Phase 6 Plan 02: Strapi/Blob Cleanup Summary

**Deleted 8 dead Strapi/Blob files, removed backend/ directory, stripped getStrapiURL/fetchStrapi from utils.ts, removed 3 Strapi/Blob remote patterns from next.config.ts, and uninstalled qs, @types/qs, @vercel/blob — zero Strapi references remain in frontend/src/**

## Performance

- **Duration:** ~4 min
- **Started:** 2026-03-30T18:36:47Z
- **Completed:** 2026-03-30T18:41:00Z
- **Tasks:** 2 of 2
- **Files modified:** 3 modified, 9 deleted (8 files + 1 directory)

## Accomplishments

- Deleted all 8 dead Strapi/Blob source files (strapi-loaders.ts, strapi-data-api.ts, strapi-image.tsx, strapi-video.tsx, error-handler.ts, optimize-avif.ts, upload-landing-images.ts, apply-optimization.ts)
- Deleted entire `backend/` directory (Strapi application code)
- Cleaned `utils.ts` from 54 lines to 6 lines — cn() only, no Strapi functions
- Removed 3 Strapi/Blob remote patterns from next.config.ts plus dangerouslyAllowLocalIP flag
- Removed qs, @types/qs from dependencies; @vercel/blob from devDependencies via direct package.json edit
- Removed 4 Strapi-related npm scripts (migrate:blob, optimize:test, optimize:apply, optimize:cleanup)
- Preserved vercel-install.mjs and migrate-strapi-to-sanity.ts (both intentional keepers)

## Task Commits

1. **Task 1: Delete dead Strapi/Blob files and backend directory** — `03e3988` (chore)
2. **Task 2: Clean utils.ts, next.config.ts, uninstall packages, remove Strapi scripts** — `0ccbb8e` (chore)

## Files Created/Modified

- `frontend/src/lib/utils.ts` — Reduced to cn() only (54 lines → 6 lines)
- `frontend/next.config.ts` — Only cdn.sanity.io remote pattern, no dangerouslyAllowLocalIP
- `frontend/package.json` — Removed qs, @types/qs, @vercel/blob, and 4 Strapi npm scripts

## Decisions Made

- Used direct package.json edit to remove qs/@types/qs/@vercel/blob because `npm uninstall` fails due to motion-plus/motion-studio using `__MOTION_DEV_TOKEN__` placeholder URLs that trigger 401 on any npm install operation. The packages were already absent from active code so no node_modules sync was needed.
- Documented pre-existing build failure (styled-components not found in @sanity/ui node_modules) confirmed via git stash test — same error exists without this plan's changes.

## Deviations from Plan

### Pre-existing Issues (out of scope, documented)

**1. Pre-existing build failure: styled-components not found in @sanity/ui**
- **Found during:** Task 2 verification (npm run build)
- **Issue:** `@sanity/ui` and `next-sanity` node_modules import `styled-components` which is not installed. Turbopack build fails with 9 errors.
- **Confirmed pre-existing:** git stash test showed identical failure before Task 2 changes
- **Action:** Documented as out-of-scope pre-existing issue, not fixed in this plan
- **Scope:** This is a dependency mismatch between Next.js 16 and Sanity packages — not caused by Strapi cleanup

---

**Total deviations:** 0 auto-fixes (cleanup executed as planned)
**Pre-existing issues documented:** 1 (styled-components build failure, out of scope)

## Issues Encountered

- `npm uninstall qs @types/qs @vercel/blob` fails because motion-plus and motion-studio use token-gated URLs (`__MOTION_DEV_TOKEN__` placeholder) that trigger 401 Unauthorized on any npm install operation. Resolved by editing package.json directly — these packages had no active importers in the codebase.

## Known Stubs

None — this plan is purely cleanup/deletion, no new features or data stubs introduced.

## Next Phase Readiness

- All Strapi source code removed from frontend/src/ — zero references remain
- backend/ directory gone — monorepo is now frontend-only
- 06-03 (env cleanup) is the final step: remove STRAPI_* and BLOB_* env vars from .env.local.example and Vercel dashboard

---
*Phase: 06-content-migration-and-cleanup*
*Completed: 2026-03-30*

## Self-Check: PASSED

- `frontend/src/lib/utils.ts`: FOUND
- `frontend/next.config.ts`: FOUND
- `frontend/package.json`: FOUND
- `.planning/phases/06-content-migration-and-cleanup/06-02-SUMMARY.md`: FOUND
- `frontend/src/data/strapi-loaders.ts`: DELETED (as required)
- `backend/`: DELETED (as required)
- Commit `03e3988`: FOUND
- Commit `0ccbb8e`: FOUND
