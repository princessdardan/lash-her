---
phase: 05-cache-revalidation
verified: 2026-03-27T00:00:00Z
status: passed
score: 6/6 must-haves verified
re_verification: false
---

# Phase 05: Cache Revalidation Verification Report

**Phase Goal:** Publishing any document in Sanity Studio causes the corresponding page to reflect updated content within seconds, without waiting for the 30-minute ISR polling interval
**Verified:** 2026-03-27
**Status:** passed
**Re-verification:** No — initial verification

## Goal Achievement

### Observable Truths

| # | Truth | Status | Evidence |
|---|-------|--------|----------|
| 1 | Every `client.fetch()` call in `loaders.ts` passes `{ next: { tags: [...] } }` as the third argument | VERIFIED | `grep -c "next: { tags:"` returns 10; all 10 loaders confirmed by inspection |
| 2 | Layout no longer uses `unstable_cache` — global/menu data fetched directly via loaders with fetch-option tags | VERIFIED | No `unstable_cache` or `getGlobalDataCached`/`getMainMenuDataCached` in `layout.tsx`; calls `loaders.getGlobalData()` and `loaders.getMainMenuData()` directly |
| 3 | POST to `/api/revalidate` with a valid Sanity signature and `_type` returns 200 and calls `revalidateTag` | VERIFIED | Route handler verifies signature, looks up tag via `TYPE_TAG_MAP`, calls `revalidateTag(tag, { expire: 0 })`, returns `new Response(null, { status: 200 })` |
| 4 | POST to `/api/revalidate` without a valid signature returns 401 | VERIFIED | Guard `if (isValidSignature !== true)` returns `new Response(null, { status: 401 })` — catches both `false` and `null` |
| 5 | POST to `/api/revalidate` with a valid signature but missing `_type` returns 400 | VERIFIED | `if (!body?._type)` returns `new Response(null, { status: 400 })` |
| 6 | `SANITY_WEBHOOK_SECRET` is validated at startup via `assertValue` — missing value crashes the app | VERIFIED | `env.ts` line 14–17: `export const webhookSecret = assertValue(process.env.SANITY_WEBHOOK_SECRET, "Missing env var: SANITY_WEBHOOK_SECRET")` |

**Score:** 6/6 truths verified

### Required Artifacts

| Artifact | Expected | Status | Details |
|----------|----------|--------|---------|
| `frontend/src/data/loaders.ts` | Cache-tagged GROQ fetches for all 10 loaders | VERIFIED | 10 loaders, each with `{ next: { tags: [...] } }` as third arg; loaders without params use `{}` as second arg |
| `frontend/src/app/(site)/layout.tsx` | Layout using direct loader calls instead of `unstable_cache` | VERIFIED | No `unstable_cache` import; calls `loaders.getGlobalData()` and `loaders.getMainMenuData()` in `SiteLayout` |
| `frontend/src/sanity/env.ts` | `SANITY_WEBHOOK_SECRET` export via `assertValue` | VERIFIED | `export const webhookSecret = assertValue(...)` present at lines 14–17 |
| `frontend/src/app/api/revalidate/route.ts` | Webhook Route Handler with HMAC verification | VERIFIED | File exists, exports `POST`, uses `parseBody`, `TYPE_TAG_MAP`, `revalidateTag(tag, { expire: 0 })` |

### Key Link Verification

| From | To | Via | Status | Details |
|------|----|-----|--------|---------|
| `frontend/src/data/loaders.ts` | `frontend/src/sanity/lib/client.ts` | `client.fetch` with tags option | VERIFIED | Pattern `client.fetch(..., {}, { next: { tags:` found 10 times |
| `frontend/src/app/api/revalidate/route.ts` | `next/cache` | `revalidateTag` call | VERIFIED | `import { revalidateTag } from "next/cache"` + `revalidateTag(tag, { expire: 0 })` at line 47 |
| `frontend/src/app/api/revalidate/route.ts` | `next-sanity/webhook` | `parseBody` for HMAC verification | VERIFIED | `import { parseBody } from "next-sanity/webhook"` + call at lines 20–23; `webhook.cjs` confirmed in `node_modules/next-sanity/dist/` |

### Data-Flow Trace (Level 4)

Not applicable for this phase. All artifacts are infrastructure components (loaders, route handler, env) — none render dynamic UI. The data-flow question is: does a published Sanity document result in a revalidated cache tag? That path is fully wired: Sanity document `_type` → webhook POST body → `TYPE_TAG_MAP` lookup → `revalidateTag(tag, { expire: 0 })` → Next.js invalidates tagged fetch responses. End-to-end delivery (Sanity webhook firing) requires human verification (see below).

### Behavioral Spot-Checks

| Behavior | Command | Result | Status |
|----------|---------|--------|--------|
| Route handler exports `POST` function | Pattern check on route.ts | `export async function POST` found | PASS |
| `parseBody` is called (not just mentioned in comment) | Line-level inspection | Call at lines 20–23; line 19 comment only | PASS |
| `revalidateTag` called with two-arg form `{ expire: 0 }` | Pattern check | `revalidateTag(tag, { expire: 0 })` at line 47 | PASS |
| `isValidSignature !== true` guard (catches null + false) | Pattern check | Line 27 confirmed | PASS |
| All 10 loaders present and all 10 cache-tagged | Node.js pattern count | 10/10 loaders found, tag count = 10 | PASS |
| No `req.json()` call before `parseBody` | Line-level inspection | Line 19 is a comment; no actual call | PASS |
| No `Response.json()` — null response bodies per security requirement | Pattern check | Absent from route.ts | PASS |
| `webhookSecret` imported from `@/sanity/env` in route handler | Grep | Line 4 of route.ts confirmed | PASS |
| `next-sanity/webhook` subpath resolvable | `node_modules` check | `webhook.cjs` exists in `next-sanity/dist/` | PASS |

### Requirements Coverage

| Requirement | Source Plan | Description | Status | Evidence |
|-------------|------------|-------------|--------|---------|
| INFRA-05 | 05-01-PLAN.md | On-demand ISR via Sanity webhook calling `/api/revalidate` with `revalidateTag()` | SATISFIED | Route handler at `app/api/revalidate/route.ts` wires `parseBody` → `TYPE_TAG_MAP` → `revalidateTag(tag, { expire: 0 })`; all 10 loaders tagged for selective invalidation |
| INFRA-06 | 05-01-PLAN.md | Webhook signature verification using shared secret (`SANITY_WEBHOOK_SECRET`) | SATISFIED | `webhookSecret` from `env.ts` passed to `parseBody`; guard `isValidSignature !== true` rejects unsigned requests with 401; `assertValue` crashes app at startup if env var missing |

No orphaned requirements — REQUIREMENTS.md traceability table maps exactly INFRA-05 and INFRA-06 to Phase 5, both claimed in plan frontmatter and both satisfied.

### Anti-Patterns Found

| File | Line | Pattern | Severity | Impact |
|------|------|---------|----------|--------|
| `frontend/src/app/(site)/layout.tsx` | 21–30 | `TODO` comments in `jsonLd` object (placeholder phone, email, address, hours) | Info | Pre-existing from prior phases; not introduced by this phase; not user-visible (JSON-LD only); does not affect cache revalidation |

No blockers or warnings introduced by this phase. The `TODO` items in `jsonLd` are pre-existing infrastructure-only placeholders that do not affect the cache revalidation goal.

### Human Verification Required

#### 1. End-to-End Webhook Delivery

**Test:** Configure `SANITY_WEBHOOK_SECRET` in `.env.local` and Vercel, create the Sanity webhook (URL: `https://<vercel-domain>/api/revalidate`, projection: `{ _type }`, with the same secret), then publish a document in Sanity Studio and verify the page reflects the change within seconds.
**Expected:** Editing and publishing a `homePage` document causes the home page to serve updated content on the next request, well under the 30-minute ISR window.
**Why human:** Requires a live Sanity webhook firing against a deployed Vercel URL. Cannot be verified from the codebase alone — depends on external service configuration and network delivery.

#### 2. Startup Crash on Missing `SANITY_WEBHOOK_SECRET`

**Test:** Start the dev server without `SANITY_WEBHOOK_SECRET` in `.env.local`.
**Expected:** Server crashes immediately with `Error: Missing env var: SANITY_WEBHOOK_SECRET`.
**Why human:** Running the dev server requires a local environment and interactive observation of startup output. TypeScript static analysis passes; the crash is a runtime behavior.

### Gaps Summary

No gaps. All 6 observable truths verified, all 4 artifacts substantive and wired, all 3 key links confirmed. INFRA-05 and INFRA-06 satisfied. One human verification item (end-to-end webhook delivery) is expected for an infrastructure phase involving external service configuration.

---

_Verified: 2026-03-27_
_Verifier: Claude (gsd-verifier)_
