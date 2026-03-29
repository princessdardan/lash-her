# Phase 6: Content Migration and Cleanup - Research

**Researched:** 2026-03-29
**Domain:** Sanity content migration from Strapi 5 REST API + codebase cleanup
**Confidence:** HIGH

---

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

- **D-01:** Migrate ALL content including historical form submissions (generalInquiry, contactForm). Not just structural content — every document in Strapi moves to Sanity for a unified record.
- **D-02:** Migration script fetches from the live Strapi Cloud REST API using the public URL + API token. No local Strapi instance needed. Script requires `STRAPI_BASE_URL` and `STRAPI_API_TOKEN` env vars.
- **D-03:** Production direct — run migration script directly against the production Sanity dataset. No staging-first dry run. Content volume is small enough that manual verification is feasible. This aligns with the big bang cutover strategy in PROJECT.md.
- **D-04:** Verification after migration: document count comparison (Strapi vs Sanity), image rendering spot-check, page content visual review. Script should output a summary report.
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

### Deferred Ideas (OUT OF SCOPE)

None — discussion stayed within phase scope.
</user_constraints>

---

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| MIG-01 | Automated migration script fetches all content from Strapi REST API, transforms to Sanity document shapes, and writes to Sanity content lake | Strapi 5 flat REST API + Sanity client v7 createOrReplace pattern |
| MIG-02 | Migration script uploads all images from Strapi to Sanity CDN and updates document references to Sanity asset IDs | `client.assets.upload()` with source metadata deduplication |
| MIG-03 | Migration script converts Strapi Blocks rich text JSON to Sanity Portable Text format | Strapi Blocks node-type mapping to PT block/span documented |
| MIG-04 | Migration script generates `_key` fields for all array items (nanoid) | nanoid v5.1.7 — `nanoid(12)` generates 12-char URL-safe key |
| MIG-05 | Migration script explicitly publishes all documents after creation (not left as drafts) | `createOrReplace` with plain `_id` (no `drafts.` prefix) creates published document directly |
| MIG-06 | Migration verified against production dataset — document counts, image rendering, and page content match source | Summary report pattern: count Strapi docs → count Sanity docs → diff |
| CLEAN-01 | Strapi backend workspace removed from `package.json` workspaces and `backend/` directory deleted | Root `package.json` has no workspaces config; `backend/` delete is a simple `rm -rf` |
| CLEAN-02 | Strapi-specific packages removed from frontend (`qs`, `@types/qs`) | `qs` only imported in `strapi-loaders.ts`; safe to remove once that file is deleted |
| CLEAN-03 | Strapi-specific code removed (old `data-api.ts`, `fetchStrapi()` in `utils.ts`, Strapi type definitions) | `utils.ts` must be EDITED not deleted — `cn()` is imported by 13 other files |
| CLEAN-04 | Vercel Blob references removed (env var, remote pattern, migration scripts) | `@vercel/blob` is in `devDependencies`, not `dependencies` |
| CLEAN-05 | Environment variables updated — Strapi vars removed, Sanity vars added to Vercel | Strapi vars live in `.env.production` and `.env.local` (not `.env.local.example`) |
</phase_requirements>

---

## Summary

Phase 6 has two distinct workstreams: (1) a one-shot migration script that moves all Strapi content into Sanity, and (2) a cleanup pass that deletes every Strapi and Vercel Blob artifact from the codebase.

The migration script must handle three non-trivial transformations: converting Strapi Blocks rich text JSON to Sanity Portable Text, uploading media from the Strapi CDN to the Sanity CDN, and generating `_key` strings for every array item. The script writes directly to the production Sanity dataset (per D-03) and creates documents as published (not drafts) by using plain IDs without a `drafts.` prefix.

The cleanup is mostly mechanical deletion, with one critical exception: `frontend/src/lib/utils.ts` cannot be deleted — it exports `cn()` which is imported by 13 files. Only `getStrapiURL()` and `fetchStrapi()` are removed from it. Similarly, `error-handler.ts` imports `TStrapiResponse` which no longer exists in `types/index.ts` — the entire file can be safely deleted (no other files import it). The root `package.json` has no workspaces configuration, so there is no workspaces entry to remove for the backend.

**Primary recommendation:** Write a single TypeScript migration script at `frontend/scripts/migrate-strapi-to-sanity.ts` that processes all content types sequentially, uploads images first to build an asset map, then creates documents with resolved references. Use `createOrReplace` with deferred visibility for bulk speed, then verify counts.

---

## Standard Stack

### Core
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| `@sanity/client` | 7.20.0 (installed) | Sanity mutations and asset uploads | Project already installed; `client.assets.upload()` + `createOrReplace` cover both upload and write needs |
| `nanoid` | 5.1.7 (current) | Generate `_key` values for array items | Tiny, secure, URL-safe; Sanity community standard for migration `_key` generation |
| `tsx` | installed (devDep) | Run TypeScript scripts directly | Already in devDependencies; used by existing scripts |
| `dotenv` | installed (devDep) | Load `.env.local` in script context | Already used by existing scripts |

### Supporting
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| `node:stream` `Readable.fromWeb` | Node.js built-in | Stream fetch response body to Sanity upload | Required for `client.assets.upload()` from URL |

### No New Dependencies Required
The migration script needs only `nanoid` as a new addition. All other capabilities (Sanity client, TypeScript runner, env loading) are already installed.

**Installation:**
```bash
cd frontend && npm install nanoid
```

**Version verification:**
```
nanoid: 5.1.7 (verified 2026-03-29 via npm view nanoid version)
@sanity/client: 7.20.0 (installed, verified)
```

---

## Architecture Patterns

### Recommended Migration Script Structure

```
frontend/scripts/
├── migrate-strapi-to-sanity.ts   # Single migration script (all content types)
├── optimize-avif.ts              # DELETE (Vercel Blob scripts)
├── upload-landing-images.ts      # DELETE
├── apply-optimization.ts         # DELETE
└── vercel-install.mjs            # KEEP (Motion dev token injection, not Strapi)
```

### Pattern 1: Document Creation as Published (MIG-05)

Sanity's draft/published model uses document ID prefix as the discriminator:
- `drafts.{id}` → draft (invisible to readers)
- `{id}` (no prefix) → published (visible to readers)

`createOrReplace` with a plain ID creates the document in published state directly. No separate publish action is needed.

```typescript
// Source: Sanity answers — https://www.sanity.io/answers/adding-and-unpublishing-documents-using-the-sanity-js-api-client-
const doc = {
  _id: 'homePage',         // No "drafts." prefix = published
  _type: 'homePage',
  title: strapiData.title,
  // ...
}
await client.createOrReplace(doc)
```

For singleton documents (homePage, contactPage, etc.), `_id === _type` (Phase 1 D: singleton documentId matches type name).

### Pattern 2: Image Upload from URL (MIG-02)

```typescript
// Source: https://www.sanity.io/learn/course/refactoring-content/uploading-assets-efficiently
import { Readable } from 'node:stream'

async function uploadImageFromUrl(
  url: string,
  sourceId: string | number
): Promise<string> {  // returns asset _id
  const response = await fetch(url)
  const asset = await client.assets.upload(
    'image',
    Readable.fromWeb(response.body),
    {
      filename: url.split('/').pop(),
      source: {
        name: 'strapi',
        id: String(sourceId),
        url,
      },
    }
  )
  return asset._id
}

// Build image reference for document
function assetRef(assetId: string) {
  return {
    _type: 'image' as const,
    asset: { _type: 'reference' as const, _ref: assetId },
  }
}
```

The `source.id` enables deduplication on re-runs: query `*[_type == "sanity.imageAsset" && source.name == "strapi"]{_id, "sourceId": source.id}` before migrating to build a cache.

Key insight from Sanity docs: "Uploading the same image binary multiple times will always result in the same ID" — deterministic hashing means accidental double-uploads are safe.

### Pattern 3: Strapi Blocks Rich Text → Portable Text (MIG-03)

Strapi Blocks field produces a JSON array of node objects. The two fields that use `"type": "blocks"` are:
- `infoSection.info` (used in trainingProgram blocks)
- `imageWithText.perks` (used in training page blocks)
- `ctaFeature.features` (used in ctaFeaturesSection blocks)

Strapi Blocks node types and their Portable Text equivalents:

| Strapi Node Type | Strapi Fields | Portable Text Block |
|-----------------|---------------|---------------------|
| `paragraph` | `children: [{type:'text', text, bold, italic}]` | `{_type:'block', style:'normal', children:[{_type:'span', text, marks:[]}]}` |
| `heading` | `level: 1-6`, children | `{_type:'block', style:'h2'/'h3', ...}` (map h1→h2, others→h3 per Phase 1 D-13) |
| `list` | `format:'ordered'/'unordered'`, children (list-items) | `{_type:'block', listItem:'bullet'/'number', level:1, ...}` |
| `quote` | children | `{_type:'block', style:'blockquote', ...}` (note: not in Sanity schema — treat as normal) |
| `link` | `url`, children | span with annotation mark `{_type:'link', _key, href: url}` |
| `image` | `image.url`, `image.alternativeText` | Upload to Sanity, add `{_type:'image', asset:{_ref}}` as PT block |
| `code` | `code`, `language` | `{_type:'block', style:'normal', children:[{text:code}]}` (no code style defined) |

**Text mark mapping** (Strapi `bold/italic` booleans → Portable Text `marks` array):
```typescript
function textMarks(node: StrapiTextNode): string[] {
  const marks: string[] = []
  if (node.bold) marks.push('strong')
  if (node.italic) marks.push('em')
  return marks
}
```

**Recommended implementation:** A `convertBlocksToPortableText(blocks: StrapiBlockNode[]): TPortableTextBlock[]` function in the script. Do not install a third-party converter — the mapping is simple enough to hand-write given the limited node types actually used.

### Pattern 4: Transaction Batching with Deferred Visibility (MIG-01)

For bulk writes, use `visibility: 'deferred'` to bypass real-time indexing during import:

```typescript
// Source: Sanity mutation docs — https://www.sanity.io/docs/http-reference/mutation
const transaction = client.transaction()
for (const doc of documents) {
  transaction.createOrReplace(doc)
}
await transaction.commit({ visibility: 'deferred' })
```

Sanity enforces ~25 requests/sec rate limit. For this project's small content volume (< 100 documents, < 50 images), a single transaction per content type is sufficient. No throttling needed.

### Pattern 5: Strapi 5 REST API Fetch (MIG-01, D-02)

Strapi 5 uses a **flat response format** — attributes are directly on the data object, not nested in `data.attributes`. The legacy `data-api.ts`/`strapi-loaders.ts` already use this correctly.

```typescript
const BASE_URL = process.env.STRAPI_BASE_URL!
const TOKEN = process.env.STRAPI_API_TOKEN!

async function fetchStrapi<T>(path: string, params = ''): Promise<T> {
  const url = `${BASE_URL}${path}${params ? '?' + params : ''}`
  const res = await fetch(url, {
    headers: { Authorization: `Bearer ${TOKEN}` },
  })
  if (!res.ok) throw new Error(`Strapi ${res.status}: ${path}`)
  const json = await res.json()
  return json.data ?? json
}

// Collection types return array; populate deeply for each type
// Single types: GET /api/home-page?populate=deep
// Collection types: GET /api/training-programs?populate=deep&pagination[pageSize]=100
// Form submissions: GET /api/contact-forms?pagination[pageSize]=1000
```

For deep population without writing component-by-component populate params, use `?populate=deep` (Strapi 5 supports depth-based population via `populate=deep` query parameter for scripts).

### Pattern 6: Menu Migration Complexity

The Strapi `menu.dropdown` component references `api::section.section` via relation (not inline component). The `section` collection has `heading` + `links[]` (menu-links). This maps to Sanity's `menuDropdown.sections` array of `menuDropdownSection` objects (inline, not references).

Migration must: fetch the menu with `?populate[MainMenuItems][on][menu.dropdown][populate][sections][populate]=true` to get sections with their links, then convert `menu.dropdown` → `menuDropdown` (with `sections` inlined) and `menu.menu-link` → `menuDirectLink`.

### Pattern 7: Training Programs Page Migration

Strapi's `trainingProgramsPage` uses a `oneToMany` relation to `trainingProgram` documents (not inline). Sanity's equivalent uses an array of references (`{_type:'reference', _ref:'trainingProgram-{id}'}`) but the Sanity schema defines `trainingPrograms` as an array of references to the `trainingProgram` document type.

Migration must create training program documents first, then create the trainingProgramsPage document with `_ref` values pointing to the created program IDs.

### Anti-Patterns to Avoid

- **Deleting `utils.ts`:** 13 components import `cn()` from it. Remove only `getStrapiURL` and `fetchStrapi` from the file.
- **Using `drafts.` prefix for IDs:** Creates invisible drafts, not published documents. Always use plain IDs.
- **Uploading images inside the document transaction:** Upload images first, build an `assetMap: Map<strapiUrl, sanityAssetId>`, then construct documents using the map. Mixing uploads and document mutations in one transaction fails.
- **Populating with `populate=*` for dynamic zones:** Strapi 5 requires `on` fragments for dynamic zones. Use `populate=deep` in migration scripts for simplicity, or explicit `on` populate for clarity.
- **Assuming `@vercel/blob` is in `dependencies`:** It's in `devDependencies`. `npm uninstall --save-dev @vercel/blob` is the correct removal command.

---

## Runtime State Inventory

This phase involves cleanup, not rename. Runtime state checklist:

| Category | Items Found | Action Required |
|----------|-------------|-----------------|
| Stored data | Strapi Cloud database (source) — all data being migrated OUT, not renamed | Migration script (Phase 6 work) |
| Live service config | Sanity Studio at `/studio` — no Strapi references | None |
| OS-registered state | None — no scheduled tasks reference Strapi | None — verified by grep of local env |
| Secrets/env vars | `NEXT_PUBLIC_STRAPI_URL`, `NEXT_PUBLIC_STRAPI_API_TOKEN` in `.env.production` and `.env.local`; `BLOB_READ_WRITE_TOKEN` in `.env.local` | Remove from `.env.production` and `.env.local`; also remove from Vercel project env vars via dashboard/CLI |
| Build artifacts | No compiled artifacts reference Strapi; `frontend/scripts/*.ts` files use `tsx` at runtime | Delete scripts (`optimize-avif.ts`, `upload-landing-images.ts`, `apply-optimization.ts`) |

**Env vars requiring Vercel dashboard/CLI update (CLEAN-05):**
- Remove: `NEXT_PUBLIC_STRAPI_URL`, `NEXT_PUBLIC_STRAPI_API_TOKEN`, `BLOB_READ_WRITE_TOKEN`
- Verify present: `NEXT_PUBLIC_SANITY_PROJECT_ID`, `NEXT_PUBLIC_SANITY_DATASET`, `NEXT_PUBLIC_SANITY_API_VERSION`, `SANITY_WRITE_TOKEN`, `SANITY_WEBHOOK_SECRET`, `RESEND_API_KEY`, `FROM_EMAIL`, `ADMIN_EMAIL`

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| `_key` generation | Custom UUID/hash | `nanoid(12)` | URL-safe, collision-resistant, standard in Sanity migrations |
| Image upload | Manual multipart/form-data | `client.assets.upload('image', stream, opts)` | Handles content-type detection, metadata extraction, deduplication |
| Strapi Blocks → PT | Install `@strapi/blocks-react-renderer` or converter libs | Hand-written mapper | The field is only used in 3 blocks (infoSection, imageWithText.perks, ctaFeature.features) with ~5 node types — a 50-line function is simpler than an external dependency |
| Published document state | Separate publish mutation | Plain `_id` with `createOrReplace` | ID without `drafts.` prefix IS the published state |

---

## Common Pitfalls

### Pitfall 1: utils.ts Deletion Breaks 13 Components
**What goes wrong:** Deleting `utils.ts` to remove Strapi functions breaks all UI components that import `cn()`.
**Why it happens:** CONTEXT.md lists `utils.ts` as a cleanup target, but it's a shared utility file.
**How to avoid:** Edit `utils.ts`, remove only `getStrapiURL` and `fetchStrapi` functions. Keep `cn()` and all imports.
**Warning signs:** TypeScript errors in `button.tsx`, `input.tsx`, `select.tsx`, `sheet.tsx`, navigation components.

### Pitfall 2: error-handler.ts Import Chain
**What goes wrong:** `error-handler.ts` imports `TStrapiResponse` from `@/types`, but `TStrapiResponse` was never defined in the Sanity-era `types/index.ts`. The file will fail to compile.
**Why it happens:** The type was a Strapi-era type that was intentionally removed from `types/index.ts` during Phase 2.
**How to avoid:** Delete `error-handler.ts` entirely — no remaining files in `src/` import from it (verified by grep).

### Pitfall 3: Root package.json Has No Workspaces
**What goes wrong:** CLEAN-01 says "remove backend from workspaces" but the root `package.json` has no `workspaces` field.
**Why it happens:** The monorepo was never formally configured with npm workspaces.
**How to avoid:** CLEAN-01 is satisfied by just deleting the `backend/` directory. No package.json edit is needed.

### Pitfall 4: @vercel/blob in devDependencies
**What goes wrong:** `npm uninstall @vercel/blob` without `--save-dev` may not correctly remove it.
**Why it happens:** `@vercel/blob` is listed under `devDependencies`, not `dependencies`.
**How to avoid:** Use `npm uninstall --save-dev @vercel/blob`.

### Pitfall 5: Strapi 5 Dynamic Zone Populate
**What goes wrong:** Using `populate=*` on pages with dynamic zones returns components without their nested fields.
**Why it happens:** Strapi 5 requires explicit `on` fragments or `populate=deep` for dynamic zones.
**How to avoid:** In the migration script, use `?populate=deep` or construct explicit populate params per endpoint.

### Pitfall 6: liveEdit Documents Don't Have Draft/Published Distinction
**What goes wrong:** `contactForm` and `generalInquiry` use `liveEdit: true` in Sanity — mutations go directly to published state. Trying to apply `drafts.` prefix will fail.
**Why it happens:** `liveEdit: true` removes the draft/published concept for these types.
**How to avoid:** For `contactForm` and `generalInquiry` documents, always write with plain IDs (no `drafts.` prefix, which is the right approach anyway per MIG-05).

### Pitfall 7: menu.dropdown sections Are Relations, Not Components
**What goes wrong:** Populating the menu with `populate=deep` may not return sections if depth is insufficient.
**Why it happens:** Strapi `menu.dropdown.sections` is a `oneToMany` relation (not a component), requiring explicit relation population.
**How to avoid:** Use explicit populate: `populate[MainMenuItems][on][menu.dropdown][populate][sections][populate][links]=true`.

### Pitfall 8: Image from photoGallery Uses `image` (multiple: true)
**What goes wrong:** `photoGallery.image` in Strapi is a multiple-media field. The migration must handle it as an array of images, each needing individual upload. The Sanity schema field is `images` (plural) — note the name difference.
**Why it happens:** Strapi field is `image`, Sanity field is `images` (mapping required).
**How to avoid:** Explicitly map `strapi.image[]` → `sanity.images[]` in the photoGallery block transformer.

### Pitfall 9: ctaFeature.features Uses Blocks Rich Text
**What goes wrong:** `ctaFeature.features` is a Strapi Blocks field, but it's nested inside `ctaFeaturesSection.features[]` which is inside the training page. Deep populate is required to get its content.
**Why it happens:** Blocks fields are deeply nested in dynamic zones.
**How to avoid:** Verify the Strapi API response shape by logging it before writing the transformer.

---

## Code Examples

### Migration Script Entry Point Structure

```typescript
// frontend/scripts/migrate-strapi-to-sanity.ts
// Source: project-specific, based on @sanity/client v7 patterns
import { createClient } from '@sanity/client'
import { nanoid } from 'nanoid'
import { Readable } from 'node:stream'
import * as dotenv from 'dotenv'

dotenv.config({ path: '.env.local' })

const client = createClient({
  projectId: process.env.NEXT_PUBLIC_SANITY_PROJECT_ID!,
  dataset: process.env.NEXT_PUBLIC_SANITY_DATASET!,
  apiVersion: '2026-03-24',
  token: process.env.SANITY_WRITE_TOKEN!,
  useCdn: false,
})

const STRAPI_BASE = process.env.STRAPI_BASE_URL!
const STRAPI_TOKEN = process.env.STRAPI_API_TOKEN!
```

### Fetch Strapi Endpoint

```typescript
async function fetchStrapi<T>(path: string): Promise<T> {
  const res = await fetch(`${STRAPI_BASE}${path}`, {
    headers: { Authorization: `Bearer ${STRAPI_TOKEN}` },
  })
  if (!res.ok) throw new Error(`Strapi ${res.status} for ${path}`)
  const json = await res.json()
  return json.data ?? json   // Strapi 5: flat response, data at top level
}
```

### Upload Image from Strapi URL

```typescript
// Source: https://www.sanity.io/learn/course/refactoring-content/uploading-assets-efficiently
const imageCache = new Map<string, string>()  // strapiUrl → sanityAssetId

async function uploadImage(url: string, strapiId: string | number): Promise<string> {
  if (imageCache.has(url)) return imageCache.get(url)!
  const res = await fetch(url)
  const asset = await client.assets.upload('image', Readable.fromWeb(res.body), {
    filename: url.split('/').pop(),
    source: { name: 'strapi', id: String(strapiId), url },
  })
  imageCache.set(url, asset._id)
  return asset._id
}

function imageRef(assetId: string) {
  return { _type: 'image' as const, asset: { _type: 'reference' as const, _ref: assetId } }
}
```

### Portable Text Converter (Strapi Blocks → PT)

```typescript
// Handles: paragraph, heading, list, link node types
// Source: Strapi Blocks format — https://thenextbit.de/en/blog/strapi-rich-text-blocks-resolver

interface StrapiTextChild { type: 'text'; text: string; bold?: boolean; italic?: boolean }
interface StrapiBlockNode {
  type: 'paragraph' | 'heading' | 'list' | 'list-item' | 'link' | 'quote' | 'code' | 'image'
  level?: number
  format?: 'ordered' | 'unordered'
  url?: string
  text?: string
  children?: Array<StrapiTextChild | StrapiBlockNode>
}

function convertBlocks(nodes: StrapiBlockNode[]): TPortableTextBlock[] {
  const blocks: TPortableTextBlock[] = []
  for (const node of nodes) {
    const key = nanoid(12)
    if (node.type === 'paragraph' || node.type === 'quote') {
      blocks.push({
        _type: 'block', _key: key, style: 'normal',
        children: flattenChildren(node.children ?? []),
        markDefs: [],
      })
    } else if (node.type === 'heading') {
      const style = node.level === 2 ? 'h2' : 'h3'
      blocks.push({
        _type: 'block', _key: key, style,
        children: flattenChildren(node.children ?? []),
        markDefs: [],
      })
    } else if (node.type === 'list') {
      const listItem = node.format === 'ordered' ? 'number' : 'bullet'
      for (const item of node.children ?? []) {
        blocks.push({
          _type: 'block', _key: nanoid(12), style: 'normal',
          listItem, level: 1,
          children: flattenChildren((item as StrapiBlockNode).children ?? []),
          markDefs: [],
        })
      }
    }
  }
  return blocks
}
```

### createOrReplace Published Document

```typescript
// Source: Sanity answers
// Plain _id (no "drafts." prefix) = published document
const transaction = client.transaction()
transaction.createOrReplace({
  _id: 'homePage',    // singleton: _id === _type
  _type: 'homePage',
  title: data.title,
  description: data.description,
  blocks: transformedBlocks,
})
await transaction.commit({ visibility: 'deferred' })
```

### Cleanup: utils.ts Edit

```typescript
// REMOVE these two functions from frontend/src/lib/utils.ts:
// - getStrapiURL()
// - fetchStrapi()
// KEEP:
// - cn() and its imports (clsx, tailwind-merge)
```

---

## Complete Field Mapping Reference

### Strapi → Sanity Document Type Map

| Strapi API Path | Strapi Kind | Sanity `_type` | Sanity `_id` | Notes |
|-----------------|-------------|----------------|-------------|-------|
| `/api/home-page` | singleType | `homePage` | `homePage` | blocks: heroSection, featuresSection |
| `/api/contact` | singleType | `contactPage` | `contactPage` | has `subTitle` field (not in other pages) |
| `/api/gallery` | singleType | `galleryPage` | `galleryPage` | |
| `/api/training` | singleType | `trainingPage` | `trainingPage` | |
| `/api/training-programs-page` | singleType | `trainingProgramsPage` | `trainingProgramsPage` | `training_programs` → `trainingPrograms` (array of refs) |
| `/api/global` | singleType | `globalSettings` | `globalSettings` | header + footer inline objects |
| `/api/main-menu` | singleType | `mainMenu` | `mainMenu` | `MainMenuItems` → `items`; complex relation populate |
| `/api/training-programs` | collectionType | `trainingProgram` | `trainingProgram-{documentId}` | slug field uses `{current: value}` in Sanity |
| `/api/contact-forms` | collectionType | `contactForm` | `contactForm-{documentId}` | `liveEdit: true`, paginate all |
| `/api/general-inquiries` | collectionType | `generalInquiry` | `generalInquiry-{documentId}` | `liveEdit: true`, paginate all |

### Blocks Field Occurrences (Rich Text Conversion Required)

| Block Type | Field | Strapi Schema Field | Strapi Type |
|------------|-------|---------------------|-------------|
| `infoSection` | `info` | `info` | blocks |
| `imageWithText` | `perks` | `perks` | blocks |
| `ctaFeature` (inside ctaFeaturesSection) | `features` | `features` | blocks |

All three map to Sanity `array` of `block` with styles [normal, h2, h3], lists [bullet, number], marks [strong, em, link].

### Image Field Occurrences (Upload Required)

| Block Type | Strapi Field | Sanity Field | Multiple |
|------------|-------------|--------------|---------|
| `heroSection` | `image` | `image` | false |
| `imageWithText` | `image` | `image` | false |
| `ctaSectionImage` | `image` | `image` | false |
| `ctaSectionVideo` | `video` | `video` | false (video, not image) |
| `photoGallery` | `image` (multiple) | `images` | true — **note field rename** |

### Cleanup Targets — Complete List

**Files to DELETE:**
- `frontend/src/components/ui/strapi-image.tsx`
- `frontend/src/components/ui/strapi-video.tsx`
- `frontend/src/data/strapi-loaders.ts`
- `frontend/src/data/strapi-data-api.ts`
- `frontend/src/lib/error-handler.ts` (no importers — safe to delete entirely)
- `frontend/scripts/optimize-avif.ts`
- `frontend/scripts/upload-landing-images.ts`
- `frontend/scripts/apply-optimization.ts`

**Files to EDIT (not delete):**
- `frontend/src/lib/utils.ts` — remove `getStrapiURL()` and `fetchStrapi()`, keep `cn()`
- `frontend/next.config.ts` — remove Strapi (`strapiapp.com`, `localhost:1337`) and Vercel Blob remote patterns; keep only `cdn.sanity.io`
- `frontend/package.json` — `npm uninstall qs @types/qs` + `npm uninstall --save-dev @vercel/blob`
- `frontend/.env.production` — remove `NEXT_PUBLIC_STRAPI_URL` and `NEXT_PUBLIC_STRAPI_API_TOKEN`
- `frontend/.env.local` — remove `NEXT_PUBLIC_STRAPI_API_URL`, `NEXT_PUBLIC_STRAPI_API_TOKEN`, `BLOB_READ_WRITE_TOKEN`
- `frontend/.env.local.example` — remove `NEXT_PUBLIC_STRAPI_URL`, `BLOB_READ_WRITE_TOKEN` references

**Directories to DELETE:**
- `backend/` (entire directory)

**Root package.json:** No edit needed — no workspaces configuration exists.

---

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Strapi v4 `data.attributes` nested response | Strapi 5 flat `data.*` directly | Strapi 5.0 (2024) | Migration script does NOT unwrap `.attributes` |
| Separate publish mutation after create | `createOrReplace` with plain ID = published | Sanity always | No extra step to publish |
| Strapi v4 `populate=*` for everything | Strapi 5 requires `on` fragments for dynamic zones | Strapi 5.0 | Script must use `populate=deep` or explicit `on` |

---

## Open Questions

1. **Which fields actually contain Strapi Blocks content in production?**
   - What we know: Schemas show `infoSection.info`, `imageWithText.perks`, `ctaFeature.features` are `"type": "blocks"`.
   - What's unclear: Whether the CMS editor has actually populated these fields with rich text (vs. plain text or empty).
   - Recommendation: In the migration script, log the raw Strapi response for these fields before writing the converter. If they're empty or plain text, the conversion is moot.

2. **Number of form submissions in Strapi**
   - What we know: contactForm and generalInquiry are collection types.
   - What's unclear: Count of historical submissions (could be 0 or hundreds).
   - Recommendation: Script should paginate with `pagination[pageSize]=100` and handle multiple pages. Strapi returns `meta.pagination` with total count.

3. **ctaSectionVideo — video upload to Sanity**
   - What we know: `cta-section-video.json` has a `video` media field (type: videos). Sanity schema has a `video` field of type `file`.
   - What's unclear: Whether any videos are actually stored in Strapi (vs. external URLs).
   - Recommendation: Handle `client.assets.upload('file', stream, {filename})` for video assets. If none exist in practice, the transformer is a no-op.

---

## Environment Availability

| Dependency | Required By | Available | Version | Fallback |
|------------|------------|-----------|---------|----------|
| Node.js | Migration script runner | Yes | 18+ (Darwin) | — |
| `tsx` | `npx tsx scripts/migrate-strapi-to-sanity.ts` | Yes (devDep installed) | in devDeps | — |
| `nanoid` | `_key` generation | Not yet | — | Install: `npm install nanoid` |
| Strapi Cloud API | MIG-01, MIG-02, MIG-03 | Yes (public URL + token in .env.production) | Strapi 5 | — |
| Sanity Write Token | MIG-01, MIG-02 | Yes (`SANITY_WRITE_TOKEN` in .env.local) | — | — |

**Missing dependencies with no fallback:**
- `nanoid` must be installed before running the migration script

**External dependencies already available:**
- Strapi Cloud URL and API token are in `.env.production`
- Sanity write token is in `.env.local`

---

## Validation Architecture

### Test Framework
| Property | Value |
|----------|-------|
| Framework | Playwright (installed, configured) |
| Config file | `frontend/playwright.config.ts` |
| Quick run command | `npx playwright test tests/homepage.spec.ts --project=chromium` |
| Full suite command | `npm test` (from frontend/) |

### Phase Requirements → Test Map

| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| MIG-01 | Script writes documents to Sanity | manual/smoke | Run script, check Sanity Studio | N/A — script output |
| MIG-02 | Images served from cdn.sanity.io | smoke | `npx playwright test tests/homepage.spec.ts --project=chromium` | Yes |
| MIG-03 | Rich text renders on training program pages | smoke | `npx playwright test tests/training-programs.spec.ts --project=chromium` | Verify exists |
| MIG-06 | Document counts match | manual | Script summary report output | N/A |
| CLEAN-01 | backend/ deleted, npm install succeeds | integration | `npm install` from root | N/A |
| CLEAN-02 | `qs` not in package.json | static | `cat frontend/package.json | grep qs` | N/A |
| CLEAN-03 | No Strapi imports in codebase | static | `grep -r "strapi" frontend/src` | N/A |
| CLEAN-04 | Vercel Blob pattern removed from next.config.ts | static | `cat frontend/next.config.ts` | N/A |
| CLEAN-05 | Strapi env vars not referenced in code | static | `grep -r "STRAPI" frontend/src` | N/A |

### Sampling Rate
- **Per task commit:** `npx playwright test tests/homepage.spec.ts --project=chromium` (30s)
- **Per wave merge:** `npm test` (full suite)
- **Phase gate:** All Playwright tests pass + script summary report shows 100% migration + `grep -r "STRAPI\|strapiapp\|vercel-storage\|@vercel/blob" frontend/src` returns empty

### Wave 0 Gaps
- [ ] Verify `frontend/tests/training-programs.spec.ts` exists — covers rich text rendering (MIG-03)
- [ ] Verify `frontend/tests/` contains tests for all 5 pages — migration correctness check

---

## Sources

### Primary (HIGH confidence)
- Sanity docs — assets upload: https://www.sanity.io/docs/apis-and-sdks/js-client-assets
- Sanity learn — uploading assets efficiently: https://www.sanity.io/learn/course/refactoring-content/uploading-assets-efficiently
- Sanity mutation docs: https://www.sanity.io/docs/http-reference/mutation
- Strapi 5 flat response format: https://docs.strapi.io/cms/migration/v4-to-v5/breaking-changes/new-response-format
- Strapi 5 REST populate: https://docs.strapi.io/cms/api/rest/populate-select
- All Strapi schemas: `backend/src/` (read directly — HIGH confidence)
- All Sanity schemas: `frontend/src/sanity/schemas/` (read directly — HIGH confidence)
- frontend `types/index.ts`, `utils.ts`, `error-handler.ts` (read directly — HIGH confidence)

### Secondary (MEDIUM confidence)
- Sanity publish via plain ID: https://www.sanity.io/answers/adding-and-unpublishing-documents-using-the-sanity-js-api-client-
- Strapi Blocks JSON format: https://thenextbit.de/en/blog/strapi-rich-text-blocks-resolver
- nanoid v5.1.7 current: https://www.npmjs.com/package/nanoid (npm registry confirmed)

### Tertiary (LOW confidence)
- `populate=deep` availability in Strapi 5 for migration scripts — assumed from Strapi 5 docs context; verify on first script run

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — packages read from installed node_modules
- Architecture: HIGH — Sanity client API verified via official docs; Strapi schemas read directly
- Strapi Blocks format: MEDIUM — format documented by community resolver; not officially verified against this specific Strapi instance
- Pitfalls: HIGH — derived from direct code inspection of all files being modified

**Research date:** 2026-03-29
**Valid until:** 2026-04-29 (Sanity and Strapi APIs are stable; nanoid is stable)
