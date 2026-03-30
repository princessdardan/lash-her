# Phase 6: Content Migration and Cleanup - Context

**Gathered:** 2026-03-29
**Status:** Ready for planning

<domain>
## Phase Boundary

Migrate all existing Strapi content (pages, settings, training programs, menu, gallery, and all historical form submissions) to the Sanity production dataset via automated scripts, verify correctness, then remove the Strapi backend and all Strapi-specific code from the codebase. After this phase, the site runs entirely on Sanity with zero Strapi remnants.

</domain>

<decisions>
## Implementation Decisions

### Migration Scope
- **D-01:** Migrate ALL content including historical form submissions (generalInquiry, contactForm). Not just structural content — every document in Strapi moves to Sanity for a unified record.
- **D-02:** Migration script fetches from the live Strapi Cloud REST API using the public URL + API token. No local Strapi instance needed. Script requires `STRAPI_BASE_URL` and `STRAPI_API_TOKEN` env vars.

### Cutover Strategy
- **D-03:** Production direct — run migration script directly against the production Sanity dataset. No staging-first dry run. Content volume is small enough that manual verification is feasible. This aligns with the big bang cutover strategy in PROJECT.md.
- **D-04:** Verification after migration: document count comparison (Strapi vs Sanity), image rendering spot-check, page content visual review. Script should output a summary report.

### Cleanup Scope
- **D-05:** Full removal — scorched earth. Delete `backend/` directory entirely, remove all `strapi-*.tsx` files, `strapi-data-api.ts`, `strapi-loaders.ts`, old image optimization scripts (`optimize-avif.ts`, `upload-landing-images.ts`, `apply-optimization.ts`), Vercel Blob references, and Strapi-specific code in `error-handler.ts` and `utils.ts`. Everything is preserved in git history.
- **D-06:** Remove Vercel Blob references: `BLOB_READ_WRITE_TOKEN` env var reference, `@vercel/blob` package (if in deps), Vercel Blob remote pattern in `next.config.ts`.
- **D-07:** Remove Strapi remote pattern from `next.config.ts` (strapiapp.com). Only `cdn.sanity.io` should remain as an image remote.

### Claude's Discretion
- Migration script architecture (single script vs. modular per content type)
- Rich text conversion implementation (Strapi Blocks JSON → Portable Text)
- Image download/upload strategy (batch vs. inline during document migration)
- `_key` generation approach (nanoid or similar)
- Whether to keep `vercel-install.mjs` script (it's for Motion dev token injection, not Strapi-related)
- Document publish strategy (create then publish, or create as published)
- Error handling and retry logic in migration script
- Whether `error-handler.ts` should be deleted entirely or just stripped of Strapi references

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Strapi Backend (source data — being deleted after migration)
- `backend/src/api/home-page/content-types/home-page/schema.json` — Homepage document structure
- `backend/src/api/contact/content-types/contact/schema.json` — Contact page structure
- `backend/src/api/gallery/content-types/gallery/schema.json` — Gallery page structure
- `backend/src/api/training/content-types/training/schema.json` — Training page structure
- `backend/src/api/training-programs-page/content-types/training-programs-page/schema.json` — Training programs overview
- `backend/src/api/training-program/content-types/training-program/schema.json` — Training program collection
- `backend/src/api/global/content-types/global/schema.json` — Global settings
- `backend/src/api/main-menu/content-types/main-menu/schema.json` — Navigation menu
- `backend/src/api/contact-form/content-types/contact-form/schema.json` — Contact form submissions
- `backend/src/api/general-inquiry/content-types/general-inquiry/schema.json` — General inquiry submissions
- `backend/src/components/layout/` — All 14 layout component schemas
- `backend/src/components/components/` — 6 reusable sub-component schemas
- `backend/src/components/menu/` — Menu component schemas
- `backend/src/services/email.ts` — Email service (already ported in Phase 4, backend copy being deleted)

### Sanity Schemas (migration target)
- `frontend/src/sanity/schemas/documents/` — All document type schemas (field names for migration mapping)
- `frontend/src/sanity/schemas/objects/` — All object type schemas (layout blocks and sub-objects)

### Frontend Files to Remove (cleanup targets)
- `frontend/src/data/strapi-loaders.ts` — Renamed old Strapi loaders (Phase 2 D-03)
- `frontend/src/data/strapi-data-api.ts` — Renamed old Strapi API client (Phase 2 D-03)
- `frontend/src/components/ui/strapi-image.tsx` — Old Strapi image component (Phase 2 D-06)
- `frontend/src/components/ui/strapi-video.tsx` — Old Strapi video component
- `frontend/src/lib/error-handler.ts` — Contains TStrapiResponse-based validation
- `frontend/src/lib/utils.ts` — Contains getStrapiURL(), getStrapiMedia() utilities
- `frontend/scripts/optimize-avif.ts` — Vercel Blob image optimization script
- `frontend/scripts/upload-landing-images.ts` — Vercel Blob upload script
- `frontend/scripts/apply-optimization.ts` — Vercel Blob optimization script

### Configuration Files
- `frontend/next.config.ts` — Remote patterns to update (remove Strapi + Vercel Blob, keep Sanity CDN)
- `frontend/package.json` — Dependencies to audit (remove qs, @types/qs, @vercel/blob if present)

### Requirements
- `.planning/REQUIREMENTS.md` — MIG-01 through MIG-06, CLEAN-01 through CLEAN-05

### Blocker from STATE.md
- "Inspect live Strapi database before writing migration script — confirm which fields actually use Blocks rich text and whether inline images appear in practice"

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- `frontend/src/sanity/lib/write-client.ts` — Write client with `SANITY_WRITE_TOKEN` for migration document creation
- `frontend/src/sanity/env.ts` — `assertValue` pattern for env var validation
- `frontend/src/sanity/schemas/` — Complete Sanity schema definitions showing exact target field names and structures
- `frontend/src/data/strapi-data-api.ts` — `fetchStrapi()` utility showing Strapi REST API call pattern (useful as reference for migration script API calls)

### Established Patterns
- **camelCase `_type` names** — `heroSection`, `contactFormLabels`, etc. (Phase 1 D-12)
- **Named exports** — All modules use named exports
- **`@/` path alias** — `frontend/src/*`
- **`server-only` import** — Write client uses this pattern
- **Portable Text block type** — Rich text fields use `block` type with styles [normal, h2, h3], lists [bullet, number], marks [strong, em, link] (Phase 1 D-13)

### Integration Points
- `frontend/scripts/` — Migration script location (alongside existing utility scripts)
- `frontend/src/sanity/lib/write-client.ts` — Sanity write client for document creation
- Root `package.json` — No workspaces config currently (backend may not be formally linked)
- `frontend/next.config.ts` — `remotePatterns` array needs Strapi and Vercel Blob entries removed

</code_context>

<specifics>
## Specific Ideas

No specific requirements — open to standard migration approaches. Key operational detail: Strapi is hosted on Strapi Cloud with a public API URL, so migration scripts can run from any environment with network access.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope.

</deferred>

---

*Phase: 06-content-migration-and-cleanup*
*Context gathered: 2026-03-29*
