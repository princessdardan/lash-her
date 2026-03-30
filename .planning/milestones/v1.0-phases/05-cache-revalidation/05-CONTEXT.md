# Phase 5: Cache Revalidation - Context

**Gathered:** 2026-03-27
**Status:** Ready for planning

<domain>
## Phase Boundary

On-demand ISR revalidation via Sanity webhooks. Publishing any document in Sanity Studio triggers a POST to `/api/revalidate` which calls `revalidateTag()` for the affected document type, causing the corresponding page(s) to reflect updated content within seconds. The current 30-minute ISR polling interval (`export const revalidate = 1800`) remains as a fallback but is no longer the primary cache invalidation mechanism. Webhook signature verification ensures only authentic Sanity requests trigger revalidation.

</domain>

<decisions>
## Implementation Decisions

### Tag Strategy
- **D-01:** Per document type cache tags. Each loader tags its fetch with the Sanity `_type` it queries (e.g., `'homePage'`, `'contactPage'`, `'galleryPage'`, `'trainingPage'`, `'trainingProgram'`, `'globalSettings'`, `'mainMenu'`). The webhook endpoint maps the incoming `_type` to the matching tag and calls `revalidateTag()`.
- **D-02:** Tags applied via fetch option — pass `{ next: { tags: ['homePage'] } }` directly in `client.fetch()` calls. No `unstable_cache` wrappers for page loaders.
- **D-03:** Migrate layout's existing `unstable_cache` wrappers for global/menu data to the fetch option tag pattern for consistency. One caching pattern across all loaders.

### Webhook Security
- **D-04:** HMAC-SHA256 signature verification using `SANITY_WEBHOOK_SECRET` env var. Sanity sends a signature header; endpoint verifies using the shared secret. No IP allowlisting — HMAC alone is sufficient.

### Revalidation Scope
- **D-05:** Webhook payload `_type` maps directly to cache tags. Each document type revalidates only its own tag — `homePage` publish revalidates tag `'homePage'`, `trainingProgram` publish revalidates tag `'trainingProgram'`, etc.
- **D-06:** `globalSettings` and `mainMenu` changes revalidate only their own tags (`'global'`/`'menu'`). Layout re-renders with updated header/footer; page data is not revalidated unless its own type changes.
- **D-07:** `trainingProgram` changes revalidate the `'trainingProgram'` tag, which covers all individual program pages. The listing page (`trainingProgramsPage`) is tagged separately.

### Error Handling
- **D-08:** HTTP status codes only — 401 for invalid/missing signature, 400 for malformed body, 200 for success. No detailed error messages in response body to prevent information leakage.
- **D-09:** Console.log key events — log document type and revalidated tags on success, log signature failures as warnings. Visible in Vercel function logs for debugging. No sensitive data logged.

### Claude's Discretion
- Exact HMAC verification implementation (crypto library choice, header name parsing)
- Whether to use Route Handler (`app/api/revalidate/route.ts`) or Pages API route
- How to structure the `_type` → tag(s) mapping (object literal, switch, etc.)
- Whether to keep `export const revalidate = 1800` as fallback or remove it

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Requirements
- `.planning/REQUIREMENTS.md` — INFRA-05 (on-demand ISR via webhook + revalidateTag) and INFRA-06 (webhook signature verification)

### Existing caching code
- `frontend/src/app/(site)/layout.tsx` — Current `unstable_cache` wrappers with `['global']` and `['menu']` tags (to be migrated)
- `frontend/src/data/loaders.ts` — All GROQ loaders that need cache tags added
- `frontend/src/app/(site)/page.tsx` — Example of current `export const revalidate = 1800` pattern (all 5 page files use this)

### Sanity client configuration
- `frontend/src/sanity/lib/client.ts` — Read client (`useCdn: true`)
- `frontend/src/sanity/env.ts` — Project ID, dataset, API version env vars

### Next.js configuration
- `frontend/next.config.ts` — Current Next.js config (no API routes exist yet)

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- `client.ts` read client: Already configured with `useCdn: true`, ready for tagged fetches
- `write-client.ts`: Write client with token — not needed for revalidation but shows env var pattern
- `env.ts`: Centralized env var access with `assertValue` — pattern to follow for `SANITY_WEBHOOK_SECRET`

### Established Patterns
- `unstable_cache` with tags in layout: Will be migrated to fetch option tags for consistency
- `export const revalidate = 1800` on all pages: Time-based ISR fallback, stays as safety net
- `client.fetch<T>(query)` in loaders: Fetch calls where `{ next: { tags: [...] } }` option will be added
- `"server-only"` import in write-client: Pattern for server-only modules

### Integration Points
- `/api/revalidate` route: New Route Handler in `frontend/src/app/api/revalidate/route.ts`
- Sanity webhook: Configured in Sanity dashboard to POST to `/api/revalidate` on document publish
- `loaders.ts`: Every loader function needs a `tags` option added to its `client.fetch()` call
- `layout.tsx`: `unstable_cache` wrappers replaced with tagged fetch in loaders

</code_context>

<specifics>
## Specific Ideas

No specific requirements — open to standard approaches. This is a well-defined infrastructure task with clear inputs (Sanity webhook) and outputs (cache invalidation).

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope.

</deferred>

---

*Phase: 05-cache-revalidation*
*Context gathered: 2026-03-27*
