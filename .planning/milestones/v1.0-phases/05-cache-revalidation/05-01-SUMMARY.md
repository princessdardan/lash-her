---
phase: 05-cache-revalidation
plan: 01
subsystem: infra
tags: [next.js, sanity, cache, revalidation, webhook, hmac, isr]

# Dependency graph
requires:
  - phase: 02-data-layer-and-image-pipeline
    provides: All 10 GROQ loaders in loaders.ts using client.fetch
  - phase: 01-schema-studio-and-infrastructure
    provides: Sanity client and env.ts assertValue pattern

provides:
  - Cache-tagged GROQ fetches on all 10 loaders (third argument to client.fetch)
  - Webhook Route Handler at /api/revalidate with HMAC-SHA256 signature verification
  - SANITY_WEBHOOK_SECRET validated at startup via assertValue in env.ts
  - Layout calling loaders directly — unstable_cache wrappers removed

affects: [vercel-deployment, content-editor-workflow, cache-invalidation]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Cache tags via fetch options: client.fetch(query, params, { next: { tags: ['tag'] } })"
    - "Empty params object required when no GROQ params: client.fetch(query, {}, { next: { tags: [...] } })"
    - "Webhook HMAC verification via parseBody from next-sanity/webhook"
    - "revalidateTag(tag, { expire: 0 }) — Next.js 16 non-deprecated form with immediate expiry"
    - "isValidSignature !== true guard (catches null and false — null when no secret provided)"

key-files:
  created:
    - frontend/src/app/api/revalidate/route.ts
  modified:
    - frontend/src/data/loaders.ts
    - frontend/src/app/(site)/layout.tsx
    - frontend/src/sanity/env.ts

key-decisions:
  - "Tags use short names: 'global' not 'globalSettings', 'menu' not 'mainMenu' — matches TYPE_TAG_MAP in route handler"
  - "getAllTrainingPrograms and getAllTrainingProgramSlugs share 'trainingProgram' tag — publishing a program revalidates both listing and individual pages"
  - "getMetaData shares 'global' tag with getGlobalData — both query globalSettings _type"
  - "unstable_cache removed from layout — fetch-level tags handle revalidation without TTL wrapper"
  - "SANITY_WEBHOOK_SECRET uses assertValue — app crashes at startup if missing, preventing silent security degradation"
  - "trainingProgramsPage intentionally absent from TYPE_TAG_MAP — no loader uses that tag; listing page revalidates via 'trainingProgram' tag"

patterns-established:
  - "Webhook route: parseBody before any req body access, null body guard, isValidSignature !== true (not === false)"
  - "TYPE_TAG_MAP: Sanity _type strings map to fetch cache tag names"

requirements-completed: [INFRA-05, INFRA-06]

# Metrics
duration: 5min
completed: 2026-03-28
---

# Phase 05 Plan 01: Cache Revalidation Summary

**On-demand ISR via Sanity webhook — all 10 GROQ loaders tagged with fetch cache tags, HMAC-verified Route Handler at /api/revalidate calling revalidateTag with immediate expiry, unstable_cache wrappers removed from layout**

## Performance

- **Duration:** 5 min
- **Started:** 2026-03-28T00:31:06Z
- **Completed:** 2026-03-28T00:35:53Z
- **Tasks:** 2
- **Files modified:** 4

## Accomplishments

- Added `{ next: { tags: [...] } }` as third argument to all 10 `client.fetch()` calls in loaders.ts — enables tag-based cache invalidation per document type
- Removed `unstable_cache` wrappers from layout.tsx — global and menu data now fetched directly via loaders with fetch-level cache tags
- Added `webhookSecret` export to env.ts via `assertValue` — app crashes at startup if `SANITY_WEBHOOK_SECRET` is not set
- Created `/api/revalidate` Route Handler with HMAC-SHA256 signature verification via `parseBody`, `TYPE_TAG_MAP` lookup, and `revalidateTag(tag, { expire: 0 })`

## Task Commits

Each task was committed atomically:

1. **Task 1: Add cache tags to all loaders and migrate layout off unstable_cache** - `0e33d22` (feat)
2. **Task 2: Create webhook Route Handler with HMAC signature verification** - `97c7a4a` (feat)

## Files Created/Modified

- `frontend/src/data/loaders.ts` — All 10 loaders now pass `{ next: { tags: ['<tag>'] } }` as the third argument to client.fetch; loaders with no GROQ params use empty object `{}` as second arg
- `frontend/src/app/(site)/layout.tsx` — Removed `unstable_cache` import and wrappers; SiteLayout now calls `loaders.getGlobalData()` and `loaders.getMainMenuData()` directly
- `frontend/src/sanity/env.ts` — Added `export const webhookSecret = assertValue(process.env.SANITY_WEBHOOK_SECRET, ...)` after projectId export
- `frontend/src/app/api/revalidate/route.ts` — New Route Handler: parseBody HMAC verification, TYPE_TAG_MAP with 7 document types, revalidateTag with { expire: 0 }, 401/400/200 null-body responses

## Decisions Made

- Tags use short canonical names (`'global'`, `'menu'`) rather than Sanity `_type` strings (`'globalSettings'`, `'mainMenu'`) — these are the cache tag identifiers used in loaders, separate from the TYPE_TAG_MAP that maps Sanity _type strings to tag names
- `trainingProgramsPage` intentionally omitted from TYPE_TAG_MAP — no loader fetches with that tag; the listing page's data comes from `getAllTrainingPrograms` which uses the `'trainingProgram'` tag

## Deviations from Plan

None — plan executed exactly as written.

## Issues Encountered

The `npm run build` verification command failed due to a pre-existing `styled-components` missing dependency in the Sanity Studio route (`/app/studio`), unrelated to this plan's changes. TypeScript-only verification (`npx tsc --noEmit --skipLibCheck`) passed cleanly for all modified and new files. This pre-existing build failure was present before this plan executed (confirmed via git stash test) and is out of scope per the deviation rules' scope boundary.

## User Setup Required

**External services require manual configuration before revalidation works end-to-end:**

1. **Generate a webhook secret:**
   ```bash
   openssl rand -hex 32
   ```

2. **Add to environment variables:**
   - `.env.local`: `SANITY_WEBHOOK_SECRET=<generated-value>`
   - Vercel dashboard: add `SANITY_WEBHOOK_SECRET` environment variable

3. **Create Sanity webhook:**
   - Go to: sanity.io/manage → Project → API → Webhooks → Create Webhook
   - URL: `https://<vercel-domain>/api/revalidate`
   - Dataset: `production`
   - Trigger on: Create, Update, Delete, Publish, Unpublish
   - Projection: `{ _type }`
   - Secret: `<same value as SANITY_WEBHOOK_SECRET>`

The app will crash at startup if `SANITY_WEBHOOK_SECRET` is not set — this is intentional to prevent silent security degradation.

## Next Phase Readiness

- On-demand cache revalidation is fully wired — content published in Sanity Studio will invalidate the corresponding Next.js cache tag within seconds
- The existing `export const revalidate = 1800` on page files remains as a safety-net fallback
- Phase 6 (Strapi removal / content migration) can proceed without cache-related concerns

---
*Phase: 05-cache-revalidation*
*Completed: 2026-03-28*

## Self-Check: PASSED

- FOUND: frontend/src/data/loaders.ts (10 cache tags added)
- FOUND: frontend/src/app/(site)/layout.tsx (unstable_cache removed)
- FOUND: frontend/src/sanity/env.ts (webhookSecret export added)
- FOUND: frontend/src/app/api/revalidate/route.ts (route handler created)
- FOUND: .planning/phases/05-cache-revalidation/05-01-SUMMARY.md
- FOUND: commit 0e33d22 (frontend sub-repo — Task 1)
- FOUND: commit 97c7a4a (frontend sub-repo — Task 2)
- FOUND: commit 5b0efc5 (root repo — planning metadata)
