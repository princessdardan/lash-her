# Phase 5: Cache Revalidation - Research

**Researched:** 2026-03-27
**Domain:** Next.js on-demand ISR, Sanity webhooks, HMAC signature verification
**Confidence:** HIGH

---

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

**Tag Strategy**
- **D-01:** Per document type cache tags. Each loader tags its fetch with the Sanity `_type` it queries (`'homePage'`, `'contactPage'`, `'galleryPage'`, `'trainingPage'`, `'trainingProgram'`, `'globalSettings'`, `'mainMenu'`). The webhook endpoint maps the incoming `_type` to the matching tag and calls `revalidateTag()`.
- **D-02:** Tags applied via fetch option — pass `{ next: { tags: ['homePage'] } }` directly in `client.fetch()` calls. No `unstable_cache` wrappers for page loaders.
- **D-03:** Migrate layout's existing `unstable_cache` wrappers for global/menu data to the fetch option tag pattern for consistency. One caching pattern across all loaders.

**Webhook Security**
- **D-04:** HMAC-SHA256 signature verification using `SANITY_WEBHOOK_SECRET` env var. Sanity sends a signature header; endpoint verifies using the shared secret. No IP allowlisting — HMAC alone is sufficient.

**Revalidation Scope**
- **D-05:** Webhook payload `_type` maps directly to cache tags. Each document type revalidates only its own tag.
- **D-06:** `globalSettings` and `mainMenu` changes revalidate only their own tags (`'global'`/`'menu'`). Layout re-renders with updated header/footer; page data is not revalidated unless its own type changes.
- **D-07:** `trainingProgram` changes revalidate the `'trainingProgram'` tag, which covers all individual program pages. The listing page (`trainingProgramsPage`) is tagged separately.

**Error Handling**
- **D-08:** HTTP status codes only — 401 for invalid/missing signature, 400 for malformed body, 200 for success. No detailed error messages in response body.
- **D-09:** Console.log key events — document type and revalidated tags on success, signature failures as warnings. No sensitive data logged.

### Claude's Discretion

- Exact HMAC verification implementation (crypto library choice, header name parsing)
- Whether to use Route Handler (`app/api/revalidate/route.ts`) or Pages API route
- How to structure the `_type` → tag(s) mapping (object literal, switch, etc.)
- Whether to keep `export const revalidate = 1800` as fallback or remove it

### Deferred Ideas (OUT OF SCOPE)

None — discussion stayed within phase scope.
</user_constraints>

---

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| INFRA-05 | On-demand ISR via Sanity webhook calling `/api/revalidate` endpoint with `revalidateTag()` — content changes go live immediately on publish | Covered by: Route Handler pattern, fetch tags in loaders, `revalidateTag({ expire: 0 })` for immediate expiry |
| INFRA-06 | Webhook signature verification using shared secret (`SANITY_WEBHOOK_SECRET`) | Covered by: `parseBody` from `next-sanity/webhook`, `SIGNATURE_HEADER_NAME = 'sanity-webhook-signature'`, HMAC-SHA256 via Web Crypto |
</phase_requirements>

---

## Summary

This phase wires Sanity's GROQ-powered webhook system to Next.js on-demand ISR revalidation. The mechanism is: Sanity Studio publishes a document → Sanity POSTs a signed payload to `/api/revalidate` → Route Handler verifies the HMAC-SHA256 signature → calls `revalidateTag()` with the matching tag → Next.js expires the cached fetch for that tag → next visitor gets fresh content without waiting for the 30-minute ISR interval.

The key pre-work is adding `{ next: { tags: ['<documentType>'] } }` to every `client.fetch()` call in `loaders.ts`, and replacing the two `unstable_cache` wrappers in `layout.tsx` with the same fetch-option tag pattern. This ensures every cached response in the system is reachable by its tag when a webhook fires.

**Critical version finding:** The project uses Next.js 16.2.1. In this version, `revalidateTag(tag)` with a single argument is **deprecated**. For webhook Route Handlers that need immediate cache expiry, the correct call is `revalidateTag(tag, { expire: 0 })`. Using `revalidateTag(tag, 'max')` would use stale-while-revalidate semantics (fine for Server Actions but not appropriate for a webhook whose purpose is immediate invalidation).

**Primary recommendation:** Use `parseBody` from `next-sanity/webhook` (already installed at v9.12.3) for signature verification. Use `revalidateTag(tag, { expire: 0 })` for immediate expiry. Use App Router Route Handler at `app/api/revalidate/route.ts`. Keep `export const revalidate = 1800` as a fallback.

---

## Standard Stack

### Core

| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| `next-sanity/webhook` | 9.12.3 (installed) | `parseBody` — reads request body, verifies HMAC-SHA256 signature | Ships with next-sanity; already installed; officially maintained by Sanity |
| `next/cache` | 16.2.1 (installed) | `revalidateTag()` — expires cached fetch entries by tag | Native Next.js API; no additional packages needed |

### Supporting

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| Node.js `crypto` (built-in) | — | Alternative HMAC impl | Only if Web Crypto API unavailable — not needed here |
| `@sanity/webhook` | separate package | Lower-level `isValidSignature` | Only needed for Pages Router (not App Router) |

**No new packages to install.** Both `next-sanity` (providing `parseBody`) and `next/cache` (providing `revalidateTag`) are already installed.

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|------------|-----------|---------|
| `parseBody` from `next-sanity/webhook` | `@sanity/webhook` `isValidSignature` directly | `parseBody` is higher-level (handles body reading + signature check in one call). `@sanity/webhook` requires manual raw-body reading. Use `parseBody` for App Router. |
| Route Handler (`route.ts`) | Pages API route (`pages/api/revalidate.ts`) | Pages API route is legacy in this codebase (App Router only project). Route Handler is the correct choice. |
| `revalidateTag(tag, { expire: 0 })` | `revalidateTag(tag, 'max')` | `'max'` = stale-while-revalidate (serves stale until next visit). `{ expire: 0 }` = immediate expiry (required for webhooks where the point is instant invalidation). |
| `fetch` option tags | `unstable_cache` wrappers | Layout currently uses `unstable_cache`. Decision D-03 is to migrate those to fetch option tags for a single consistent pattern. |

**Installation:** No new packages needed.

---

## Architecture Patterns

### Recommended Project Structure

```
frontend/src/app/
├── api/
│   └── revalidate/
│       └── route.ts          # New: POST handler for Sanity webhooks
└── (site)/
    ├── layout.tsx             # Modify: remove unstable_cache, add tags to loaders
    ├── page.tsx               # Unchanged (revalidate=1800 stays as fallback)
    ├── contact/page.tsx       # Unchanged
    ├── gallery/page.tsx       # Unchanged
    ├── training/page.tsx      # Unchanged
    └── training-programs/[slug]/page.tsx  # Unchanged

frontend/src/
├── data/
│   └── loaders.ts             # Modify: add { next: { tags: [...] } } to every fetch
├── config/                    # Possibly: centralize tag constants
└── sanity/
    └── env.ts                 # Modify: add SANITY_WEBHOOK_SECRET via assertValue
```

### Pattern 1: Fetch Option Tags on `client.fetch()`

**What:** Pass `{ next: { tags: ['<documentType>'] } }` as the third argument to every `client.fetch()` call in `loaders.ts`.

**When to use:** All page-level loaders where on-demand revalidation is needed.

**Example:**
```typescript
// Source: Next.js docs + next-sanity client.fetch signature
// frontend/src/data/loaders.ts

async function getHomePageData(): Promise<THomePage | null> {
  const query = groq`*[_type == "homePage"][0]{ ... }`;
  return client.fetch<THomePage | null>(query, {}, {
    next: { tags: ['homePage'] },
  });
}

async function getGlobalData(): Promise<TGlobalSettings | null> {
  const query = groq`*[_type == "globalSettings"][0]{ ... }`;
  return client.fetch<TGlobalSettings | null>(query, {}, {
    next: { tags: ['global'] },
  });
}

// Parameterized loaders — params go in second arg, options in third
async function getTrainingProgramBySlug(slug: string): Promise<TTrainingProgram | null> {
  const query = groq`*[_type == "trainingProgram" && slug.current == $slug][0]{ ... }`;
  return client.fetch<TTrainingProgram | null>(query, { slug }, {
    next: { tags: ['trainingProgram'] },
  });
}
```

**Important:** `client.fetch()` in `@sanity/client` v7+ accepts options as the third argument. The second argument is GROQ params (`{}`  if none). Passing options as the second arg when params are empty is a common mistake — it will treat them as query params and silently ignore the `next` config.

### Pattern 2: `unstable_cache` Migration in `layout.tsx`

**What:** Replace the two `unstable_cache` wrappers with direct loader calls that use fetch option tags.

**When to use:** The layout loaders for `globalData` and `mainMenu`.

**Example:**
```typescript
// Source: Next.js ISR docs — fetch option tags replace unstable_cache tags
// frontend/src/app/(site)/layout.tsx  (BEFORE → AFTER)

// BEFORE (remove these):
const getGlobalDataCached = unstable_cache(
  async () => loaders.getGlobalData(),
  ['global-data'],
  { revalidate: 3600, tags: ['global'] }
);

// AFTER (use loader directly — tags are now on the fetch in loaders.ts):
const globalData = await loaders.getGlobalData();
const mainMenuData = await loaders.getMainMenuData();
```

After migration, remove `unstable_cache` import from `layout.tsx`.

### Pattern 3: Route Handler with `parseBody` + `revalidateTag`

**What:** App Router POST handler at `app/api/revalidate/route.ts`. Uses `parseBody` from `next-sanity/webhook` for signature verification. Maps incoming `_type` to a cache tag and calls `revalidateTag`.

**When to use:** This is the only endpoint file for the entire phase.

```typescript
// Source: next-sanity/webhook module source (verified in installed package)
// + Next.js 16 revalidateTag API reference (official docs)
// frontend/src/app/api/revalidate/route.ts

import { revalidateTag } from 'next/cache';
import type { NextRequest } from 'next/server';
import { parseBody } from 'next-sanity/webhook';

// Map Sanity _type to cache tag
// Uses the document types defined in the schema (Phase 1)
const TYPE_TAG_MAP: Record<string, string> = {
  homePage: 'homePage',
  contactPage: 'contactPage',
  galleryPage: 'galleryPage',
  trainingPage: 'trainingPage',
  trainingProgramsPage: 'trainingProgramsPage',
  trainingProgram: 'trainingProgram',
  globalSettings: 'global',
  mainMenu: 'menu',
};

export async function POST(req: NextRequest): Promise<Response> {
  const secret = process.env.SANITY_WEBHOOK_SECRET;

  const { body, isValidSignature } = await parseBody<{ _type: string }>(
    req,
    secret
  );

  if (isValidSignature === false) {
    console.warn('[revalidate] Invalid webhook signature');
    return new Response(null, { status: 401 });
  }

  if (!body?._type) {
    console.warn('[revalidate] Webhook body missing _type');
    return new Response(null, { status: 400 });
  }

  const tag = TYPE_TAG_MAP[body._type];

  if (!tag) {
    // Unknown type — not an error, just nothing to revalidate
    console.log(`[revalidate] Unknown _type: ${body._type} — skipping`);
    return new Response(null, { status: 200 });
  }

  revalidateTag(tag, { expire: 0 });
  console.log(`[revalidate] Revalidated tag '${tag}' for _type '${body._type}'`);

  return new Response(null, { status: 200 });
}
```

### Pattern 4: `SANITY_WEBHOOK_SECRET` in `env.ts`

**What:** Add the webhook secret to the centralized env file using the project's `assertValue` pattern.

```typescript
// Source: Existing pattern in frontend/src/sanity/env.ts
export const webhookSecret = assertValue(
  process.env.SANITY_WEBHOOK_SECRET,
  "Missing env var: SANITY_WEBHOOK_SECRET"
);
```

**Decision:** The route handler accesses `process.env.SANITY_WEBHOOK_SECRET` directly (following the write-client pattern where token is read inline) OR uses the centralized helper. Either approach is fine; the centralized approach is more consistent with the project's style.

### Tag Reference: `_type` → Cache Tag Mapping

| Sanity `_type` | Cache Tag | Loader(s) using it |
|---------------|-----------|-------------------|
| `homePage` | `'homePage'` | `getHomePageData` |
| `contactPage` | `'contactPage'` | `getContactPageData` |
| `galleryPage` | `'galleryPage'` | `getGalleryPageData` |
| `trainingPage` | `'trainingPage'` | `getTrainingsPageData` |
| `trainingProgramsPage` | `'trainingProgramsPage'` | *(no current loader — may be unused)* |
| `trainingProgram` | `'trainingProgram'` | `getTrainingProgramBySlug`, `getAllTrainingPrograms`, `getAllTrainingProgramSlugs` |
| `globalSettings` | `'global'` | `getGlobalData`, `getMetaData` |
| `mainMenu` | `'menu'` | `getMainMenuData` |

**Note:** `getMetaData` queries `globalSettings` — it must also receive tag `'global'`. Both loaders that query the same `_type` must share the same tag.

### Anti-Patterns to Avoid

- **Single-argument `revalidateTag(tag)`:** Deprecated in Next.js 16. TypeScript will not catch this unless strict signatures are enforced. Always use `revalidateTag(tag, { expire: 0 })` in Route Handlers.
- **Options in second arg position when params are empty:** `client.fetch(query, { next: { tags: [...] } })` silently treats tags as GROQ params. Always use `client.fetch(query, {}, { next: { tags: [...] } })`.
- **Leaving `SANITY_WEBHOOK_SECRET` unset:** `parseBody` returns `isValidSignature: null` when no secret is provided — this is NOT the same as false. The endpoint would accept all requests. Guard with `assertValue` in env.ts to fail loudly at startup if missing.
- **Returning error details in response body:** D-08 prohibits this. Return `new Response(null, { status: 401 })` not `Response.json({ message: 'Invalid signature' })`.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Webhook signature verification | Custom HMAC logic using Node's `crypto` module | `parseBody` from `next-sanity/webhook` | Handles raw body reading, header extraction, HMAC-SHA256 comparison, and timestamp validation in one call. Manual implementation risks subtle timing attack vulnerabilities. |
| Cache tag invalidation | Custom fetch cache-busting (cache-control headers, URL param tricks) | `revalidateTag` from `next/cache` | Native Next.js API that integrates with the ISR cache layer correctly. Custom solutions don't reach the Next.js data cache. |
| Webhook body parsing | `req.json()` before signature check | `parseBody` from `next-sanity/webhook` | Sanity's signature is over the raw bytes. Calling `req.json()` first consumes the stream; `parseBody` reads `req.text()` internally. Reading the body as JSON first invalidates the signature check. |

**Key insight:** The most dangerous DIY mistake is calling `req.json()` before signature verification. HMAC verification requires the raw string body that Sanity signed — JSON parsing can alter whitespace/ordering, causing all signatures to fail. `parseBody` reads the raw body text internally before parsing, so it handles this correctly.

---

## Common Pitfalls

### Pitfall 1: `client.fetch` options in wrong argument position

**What goes wrong:** Tags are silently ignored; all pages get invalidated (or none).
**Why it happens:** `@sanity/client` v7 `fetch()` signature is `(query, params, options)`. Passing `{ next: { tags: [...] } }` as the second argument treats it as GROQ params.
**How to avoid:** Always use three arguments: `client.fetch(query, {}, { next: { tags: [...] } })` for loaders with no GROQ params, and `client.fetch(query, { slug }, { next: { tags: [...] } })` for parameterized ones.
**Warning signs:** Revalidation fires in Vercel logs but page content doesn't refresh.

### Pitfall 2: `revalidateTag` single-argument deprecation

**What goes wrong:** TypeScript warning at build time; behavior may change in a future Next.js minor version.
**Why it happens:** Next.js 16 introduced `{ expire: 0 }` for immediate expiry and `profile='max'` for SWR semantics. The old no-profile form is deprecated.
**How to avoid:** Use `revalidateTag(tag, { expire: 0 })` in Route Handlers. This matches D-08 (immediate expiry for webhook-triggered revalidation).
**Warning signs:** TypeScript errors on `revalidateTag` signature.

### Pitfall 3: `parseBody` 3-second delay

**What goes wrong:** Webhook handler takes 3+ seconds to respond; Sanity may log a slow delivery warning.
**Why it happens:** `parseBody` from `next-sanity/webhook` (verified in installed v9.12.3 source) includes `await new Promise(resolve => setTimeout(resolve, 3000))` when `waitForContentLakeEventualConsistency = true` (the default). This delay exists to let Sanity's eventual consistency propagate before revalidation fires.
**How to avoid:** This delay is intentional and correct — do not disable it. The 3-second wait ensures the new content is readable when the revalidated page renders. Sanity webhooks have a 30-second timeout, so 3 seconds is fine.
**Warning signs:** Vercel function logs show handler duration of ~3 seconds — this is expected behavior, not a bug.

### Pitfall 4: Missing `SANITY_WEBHOOK_SECRET` in production

**What goes wrong:** `parseBody` returns `isValidSignature: null` (not `false`) when no secret is passed. If the endpoint doesn't distinguish `null` from `true`, all unsigned requests succeed.
**Why it happens:** `parseBody` treats `secret = undefined` as "no verification" and returns `null` instead of failing.
**How to avoid:** Add `SANITY_WEBHOOK_SECRET` to `env.ts` via `assertValue`. The app will crash loudly at startup if it's missing, preventing silent security degradation.
**Warning signs:** Production revalidations fire on every request including non-Sanity traffic.

### Pitfall 5: `unstable_cache` tags not invalidated by `revalidateTag`

**What goes wrong:** After phase completion, layout data (header/footer) doesn't refresh on webhook.
**Why it happens:** `unstable_cache` tags CAN be invalidated by `revalidateTag`, but the layout currently uses `unstable_cache` with a `revalidate: 3600` time. After migrating to fetch option tags (D-03), `revalidateTag('global')` will work as expected.
**How to avoid:** Complete the `unstable_cache` → fetch option tag migration in `layout.tsx` as part of this phase.
**Warning signs:** Webhook fires and page content updates, but header/footer logo text or nav items remain stale.

### Pitfall 6: `trainingProgramsPage` type has no loader

**What goes wrong:** Mapping `trainingProgramsPage → 'trainingProgramsPage'` in the TYPE_TAG_MAP is harmless, but no loader tags fetches with `'trainingProgramsPage'`, so revalidation of that tag is a no-op.
**Why it happens:** Per D-07, the listing page is a separate tag from individual programs. But the listing page (`training-programs/`) currently fetches via `getAllTrainingPrograms` which will be tagged `'trainingProgram'`. If a `trainingProgramsPage` document exists in the schema with a separate loader, add it. If not, map `trainingProgram` publish to also revalidate the listing page implicitly (since the listing loader will share the `'trainingProgram'` tag).
**How to avoid:** Verify which loader backs the training programs listing page. Tag it consistently.

---

## Code Examples

### Adding tags to a no-param loader

```typescript
// Source: Verified against @sanity/client v7 fetch() signature
async function getHomePageData(): Promise<THomePage | null> {
  const query = groq`*[_type == "homePage"][0]{ ... }`;
  return client.fetch<THomePage | null>(query, {}, {
    next: { tags: ['homePage'] },
  });
}
```

### Adding tags to a parameterized loader

```typescript
// Source: Verified against @sanity/client v7 fetch() signature
async function getTrainingProgramBySlug(slug: string): Promise<TTrainingProgram | null> {
  const query = groq`*[_type == "trainingProgram" && slug.current == $slug][0]{ ... }`;
  return client.fetch<TTrainingProgram | null>(query, { slug }, {
    next: { tags: ['trainingProgram'] },
  });
}
```

### `parseBody` internals (verified from installed package source)

```typescript
// Source: next-sanity/dist/webhook.js — installed at v9.12.3
// SIGNATURE_HEADER_NAME = 'sanity-webhook-signature'
// parseBody reads req.text() (not req.json()), then JSON.parses after verification
// Includes 3-second delay for Content Lake eventual consistency (default: true)
async function parseBody(req, secret, waitForContentLakeEventualConsistency = true) {
  const signature = req.headers.get('sanity-webhook-signature');
  if (!signature) return { body: null, isValidSignature: null };
  const body = await req.text();
  const validSignature = secret ? await isValidSignature(body, signature, secret.trim()) : null;
  if (validSignature !== false && waitForContentLakeEventualConsistency) {
    await new Promise(resolve => setTimeout(resolve, 3000));
  }
  return { body: body.trim() ? JSON.parse(body) : null, isValidSignature: validSignature };
}
```

### Full Route Handler (ready to implement)

```typescript
// Source: Official pattern verified against Next.js 16.2.1 docs + next-sanity/webhook v9
// frontend/src/app/api/revalidate/route.ts
import { revalidateTag } from 'next/cache';
import type { NextRequest } from 'next/server';
import { parseBody } from 'next-sanity/webhook';

const TYPE_TAG_MAP: Record<string, string> = {
  homePage: 'homePage',
  contactPage: 'contactPage',
  galleryPage: 'galleryPage',
  trainingPage: 'trainingPage',
  trainingProgram: 'trainingProgram',
  globalSettings: 'global',
  mainMenu: 'menu',
};

export async function POST(req: NextRequest): Promise<Response> {
  const { body, isValidSignature } = await parseBody<{ _type: string }>(
    req,
    process.env.SANITY_WEBHOOK_SECRET
  );

  if (isValidSignature === false) {
    console.warn('[revalidate] Invalid webhook signature');
    return new Response(null, { status: 401 });
  }

  if (!body?._type) {
    console.warn('[revalidate] Webhook body missing _type');
    return new Response(null, { status: 400 });
  }

  const tag = TYPE_TAG_MAP[body._type];

  if (!tag) {
    console.log(`[revalidate] Unhandled _type: ${body._type} — no-op`);
    return new Response(null, { status: 200 });
  }

  revalidateTag(tag, { expire: 0 });
  console.log(`[revalidate] tag='${tag}' _type='${body._type}'`);

  return new Response(null, { status: 200 });
}
```

### `layout.tsx` migration (before / after)

```typescript
// BEFORE — remove these lines:
import { unstable_cache } from 'next/cache';

const getGlobalDataCached = unstable_cache(
  async () => loaders.getGlobalData(),
  ['global-data'],
  { revalidate: 3600, tags: ['global'] }
);

const getMainMenuDataCached = unstable_cache(
  async () => loaders.getMainMenuData(),
  ['main-menu-data'],
  { revalidate: 3600, tags: ['menu'] }
);

// In SiteLayout():
const globalData = await getGlobalDataCached();
const mainMenuData = await getMainMenuDataCached();

// AFTER — replace with direct loader calls (tags now live on client.fetch in loaders.ts):
const globalData = await loaders.getGlobalData();
const mainMenuData = await loaders.getMainMenuData();
```

### Sanity Dashboard Webhook Configuration

```
URL:           https://<your-vercel-domain>/api/revalidate
Dataset:       production (and staging if needed)
HTTP Method:   POST
Trigger on:    Create, Update, Delete, Publish, Unpublish
Filter:        (none — revalidate on any document change)
Projection:    { _type }
Secret:        <same value as SANITY_WEBHOOK_SECRET env var>
Status:        Enabled
```

---

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Single-arg `revalidateTag(tag)` | `revalidateTag(tag, { expire: 0 })` | Next.js 16 | Old form deprecated; TypeScript errors if strict |
| `unstable_cache` for layout data | Fetch option tags on `client.fetch()` | Decision D-03 | One consistent caching pattern across all loaders |
| Pages API route for webhooks | App Router Route Handler | next-sanity v9 | `next-sanity/webhook` is App Router-only in v9 |

**Deprecated/outdated:**
- `revalidateTag(tag)` single-argument: Deprecated in Next.js 16. Works now but may be removed.
- `unstable_cache` wrappers in `layout.tsx`: Will be replaced as part of this phase per D-03.
- `res.revalidate()` (Pages Router): Not applicable — App Router project.

---

## Open Questions

1. **`trainingProgramsPage` document type — does it have its own loader?**
   - What we know: Schema exists for `trainingProgramsPage` (from REQUIREMENTS.md SCHEMA-01), but no loader named `getTrainingProgramsPageData` exists in `loaders.ts`. The listing page at `training-programs/` likely uses `getAllTrainingPrograms` or `getTrainingsPageData`.
   - What's unclear: Which loader backs the training programs listing page (`/training-programs`), and whether `trainingProgramsPage` documents are published in Sanity.
   - Recommendation: During implementation, grep the training-programs listing page component to identify which loaders it calls. Tag all of them with `'trainingProgram'` since that's the content they display. If `trainingProgramsPage` is a singleton with its own loader, add it to `TYPE_TAG_MAP`.

2. **`isValidSignature: null` guard in route handler**
   - What we know: `parseBody` returns `null` (not `false`) when no secret is configured.
   - What's unclear: Should the handler treat `null` as valid (permissive dev mode) or as an error?
   - Recommendation: Guard explicitly with `if (isValidSignature !== true)` rather than `if (isValidSignature === false)` in production. This treats missing secret as an auth failure. Use the `assertValue` pattern in `env.ts` to prevent missing secret at startup.

---

## Environment Availability

| Dependency | Required By | Available | Version | Fallback |
|------------|------------|-----------|---------|----------|
| `next-sanity/webhook` (`parseBody`) | Signature verification | Yes | 9.12.3 | — |
| `next/cache` (`revalidateTag`) | Cache invalidation | Yes | 16.2.1 | — |
| Sanity Dashboard access | Webhook creation | Unknown — user must verify | — | Cannot configure without dashboard access |
| `SANITY_WEBHOOK_SECRET` env var | Route Handler auth | Not yet set | — | Phase cannot complete without this |

**Missing dependencies with no fallback:**
- `SANITY_WEBHOOK_SECRET` env var — must be generated and added to `.env.local` and Vercel environment variables before the webhook can function. A plan task should document this as a required manual step.
- Sanity Dashboard access — webhook configuration is a manual step in `sanity.io/manage`. Cannot be automated via code. Must be documented as a plan task.

**Missing dependencies with fallback:**
- None.

---

## Validation Architecture

### Test Framework

| Property | Value |
|----------|-------|
| Framework | Playwright 1.58.2 |
| Config file | `frontend/playwright.config.ts` |
| Quick run command | `cd frontend && npx playwright test tests/` |
| Full suite command | `cd frontend && npm run test` |

### Phase Requirements → Test Map

| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| INFRA-05 | Webhook POST triggers revalidateTag for correct document type | Integration (manual) | N/A — requires live Vercel + Sanity | No |
| INFRA-05 | Tags are present on fetch calls in loaders (smoke) | Smoke (manual) | Verify via `NEXT_PRIVATE_DEBUG_CACHE=1 npm run dev` | No |
| INFRA-06 | Route handler returns 401 for invalid signature | API unit | `curl -X POST http://localhost:3000/api/revalidate` | No — Wave 0 |
| INFRA-06 | Route handler returns 401 for missing signature | API unit | manual curl test | No — Wave 0 |
| INFRA-06 | Route handler returns 400 for missing `_type` | API unit | manual curl test | No — Wave 0 |
| INFRA-06 | Route handler returns 200 for valid request | API unit | manual curl test | No — Wave 0 |

**Note:** The Playwright test suite (`frontend/tests/`) currently mocks Strapi endpoints. It does not cover Next.js API routes or cache behavior. The most practical validation for this phase is **manual verification** per the success criteria: confirm Vercel function logs show the webhook firing, confirm 401 is returned for invalid signatures (testable via curl), and confirm a hard refresh shows updated content after a Sanity publish.

### Sampling Rate

- **Per task commit:** `cd frontend && npm run build` (confirms no TypeScript errors in new route file)
- **Per wave merge:** `cd frontend && npm run build && npm run lint`
- **Phase gate:** Manual end-to-end test per success criteria before `/gsd:verify-work`

### Wave 0 Gaps

- [ ] No automated tests exist for the `/api/revalidate` Route Handler — manual curl verification is the primary validation
- [ ] Playwright fixtures still mock Strapi endpoints — these don't need updating for this phase (cache behavior tests require a live Sanity connection)

*(Existing Playwright tests do not cover this phase's requirements. Manual verification is the intended validation path for cache-layer changes.)*

---

## Project Constraints (from CLAUDE.md)

The project uses a global CLAUDE.md (user-level). No project-level `./CLAUDE.md` was found in the working directory.

Global directives that apply to this phase:

| Directive | Application |
|-----------|-------------|
| TypeScript strict: explicit types, no `any`, strict null checks | Route Handler must type the `body` payload: `parseBody<{ _type: string }>`. No `any` on the error catch. |
| `interface` over `type` for object shapes | `TYPE_TAG_MAP` is `Record<string, string>` — not an interface; this is a type alias, which is correct. |
| Named exports over default exports | Route Handler exports named `POST` function — correct. |
| Prefer early returns | Route Handler should return early on invalid signature, then on missing body, before reaching `revalidateTag`. |
| Error handling: throw typed errors, don't swallow | Don't catch and silence errors from `parseBody`. Let unexpected errors propagate (500 from Next.js default). |
| Package manager: npm | `npm install` if any new package were needed (none are). |

---

## Sources

### Primary (HIGH confidence)

- Next.js 16.2.1 official docs — `revalidateTag` API reference (fetched directly, dated 2026-03-25)
- Next.js 16.2.1 official docs — ISR guide with on-demand revalidation patterns (fetched directly, dated 2026-03-25)
- `next-sanity/dist/webhook.js` — Installed package source (v9.12.3) — `parseBody` implementation, `SIGNATURE_HEADER_NAME`, 3-second delay behavior (verified by direct inspection)

### Secondary (MEDIUM confidence)

- [Sanity Guides: Webhooks and On-demand Revalidation in Next.js](https://www.sanity.io/guides/sanity-webhooks-and-on-demand-revalidation-in-nextjs) — Route Handler pattern + Sanity dashboard config
- [next-sanity v8 → v9 migration guide](https://github.com/sanity-io/next-sanity/blob/main/packages/next-sanity/MIGRATE-v8-to-v9.md) — confirms `next-sanity/webhook` is App Router-only in v9; Pages Router requires `@sanity/webhook` directly
- [Next.js `revalidateTag` deprecation discussion](https://github.com/vercel/next.js/discussions/84805) — `updateTag` vs `revalidateTag`, `{ expire: 0 }` for immediate expiry

### Tertiary (LOW confidence)

- WebSearch results on HMAC header name `sanity-webhook-signature` — confirmed by direct package source inspection (elevated to HIGH)

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — `next-sanity/webhook` is installed and source-inspected; `next/cache` is native
- Architecture: HIGH — Next.js docs fetched directly; `parseBody` source verified
- Pitfalls: HIGH — three pitfalls verified from installed package source or official docs; two from cross-referenced WebSearch

**Research date:** 2026-03-27
**Valid until:** 2026-04-27 (stable API; Next.js 16 recently released, unlikely to change within 30 days)
