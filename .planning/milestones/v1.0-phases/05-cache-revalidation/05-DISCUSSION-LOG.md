# Phase 5: Cache Revalidation - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-03-27
**Phase:** 05-cache-revalidation
**Areas discussed:** Tag strategy, Webhook security, Revalidation scope, Error handling

---

## Tag Strategy

### Q1: How granular should cache tags be?

| Option | Description | Selected |
|--------|-------------|----------|
| Per document type | Tags like 'homePage', 'contactPage', 'trainingProgram', 'global', 'menu'. Webhook maps _type to tag, revalidates only pages using that type. | :heavy_check_mark: |
| Single global tag | One tag like 'sanity-content' on everything. Any publish revalidates the entire site. | |
| Per document ID | Tags like 'doc-homePage', 'doc-trainingProgram-slug'. Most precise, overkill for this content volume. | |

**User's choice:** Per document type
**Notes:** Good balance of precision and simplicity for a small site.

### Q2: Should loaders add tags inline (fetch option) or via unstable_cache wrappers?

| Option | Description | Selected |
|--------|-------------|----------|
| Fetch option tags | Pass { next: { tags: ['homePage'] } } directly in client.fetch() calls. | :heavy_check_mark: |
| unstable_cache wrappers | Wrap each loader in unstable_cache() like the layout does for global/menu. | |

**User's choice:** Fetch option tags
**Notes:** Simpler, no wrapper functions needed.

### Q3: Should layout's existing unstable_cache wrappers be migrated?

| Option | Description | Selected |
|--------|-------------|----------|
| Yes, migrate | Move layout's global and menu data fetching to use fetch option tags. One consistent pattern. | :heavy_check_mark: |
| No, keep both | Leave layout's unstable_cache as-is, use fetch option tags for page loaders. | |

**User's choice:** Yes, migrate
**Notes:** One consistent pattern across all loaders.

---

## Webhook Security

### Q1: How should the /api/revalidate endpoint verify webhook authenticity?

| Option | Description | Selected |
|--------|-------------|----------|
| HMAC signature | Sanity sends an HMAC-SHA256 signature header. Endpoint verifies using SANITY_WEBHOOK_SECRET env var. | :heavy_check_mark: |
| Shared secret header | Simple authorization header check. Easier but less secure. | |
| Both HMAC + IP allowlist | HMAC verification plus restrict to Sanity's webhook IP ranges. Maximum security but maintenance burden. | |

**User's choice:** HMAC signature
**Notes:** Industry standard, no external dependencies.

---

## Revalidation Scope

### Q1: Should changing globalSettings or mainMenu revalidate ALL pages?

| Option | Description | Selected |
|--------|-------------|----------|
| Layout tags only | Revalidate tags 'global' and 'menu'. Layout re-renders, pages themselves aren't revalidated. | :heavy_check_mark: |
| All page tags too | Also revalidate all page-level tags. Belt-and-suspenders. | |

**User's choice:** Layout tags only
**Notes:** Next.js re-renders the layout which feeds updated header/footer to all pages.

### Q2: Should the endpoint handle trainingProgram changes (collection type)?

| Option | Description | Selected |
|--------|-------------|----------|
| Revalidate type tag | Tag all training program pages with 'trainingProgram'. Any publish revalidates all of them. | :heavy_check_mark: |
| Revalidate type + listing | Revalidate both 'trainingProgram' and 'trainingProgramsPage'. | |
| You decide | Claude picks best approach during implementation. | |

**User's choice:** Revalidate type tag
**Notes:** Simple approach, only ~4 programs.

---

## Error Handling

### Q1: How should the endpoint respond to invalid or unauthorized requests?

| Option | Description | Selected |
|--------|-------------|----------|
| Status codes only | 401 for invalid signature, 400 for malformed body, 200 for success. No error messages. | :heavy_check_mark: |
| Status codes + messages | Same codes but include JSON error messages. | |
| You decide | Claude picks based on security best practices. | |

**User's choice:** Status codes only
**Notes:** Prevents information leakage.

### Q2: Should the endpoint log revalidation events?

| Option | Description | Selected |
|--------|-------------|----------|
| Console.log key events | Log document type and revalidated tags on success, signature failures as warnings. | :heavy_check_mark: |
| Silent operation | No logging, rely on HTTP status codes. | |
| You decide | Claude picks the logging level. | |

**User's choice:** Console.log key events
**Notes:** Visible in Vercel function logs for debugging.

---

## Claude's Discretion

- HMAC verification implementation details (crypto library, header parsing)
- Route Handler vs Pages API route choice
- _type → tag mapping structure
- Whether to keep revalidate = 1800 as fallback

## Deferred Ideas

None — discussion stayed within phase scope.
