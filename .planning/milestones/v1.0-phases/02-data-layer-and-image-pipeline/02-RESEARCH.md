# Phase 2: Data Layer and Image Pipeline - Research

**Researched:** 2026-03-25
**Domain:** GROQ queries, next-sanity client, @sanity/image-url, Next.js image pipeline, TypeScript type rewrite
**Confidence:** HIGH

---

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

**GROQ Loader Organization**
- **D-01:** Keep the single `loaders.ts` file pattern — one file exporting a `loaders` object with all functions. GROQ queries written inline in each function.
- **D-02:** Import the Sanity read client directly from `@/sanity/lib/client` in each loader. No wrapper function — call `client.fetch<T>(query, params)` directly. Matches next-sanity conventions.
- **D-03:** Old Strapi files kept during Phase 2: rename `loaders.ts` to `strapi-loaders.ts` and `data-api.ts` to `strapi-data-api.ts`. New Sanity `loaders.ts` created fresh. Clean removal in Phase 6.

**Image Component Design**
- **D-04:** New `sanity-image.tsx` component accepts the raw Sanity image object directly (asset ref, hotspot, crop fields). Uses `@sanity/image-url` builder internally to generate URLs. Layout components pass the Sanity image field as-is — no manual URL extraction at the caller site.
- **D-05:** Full hotspot/crop support enabled. The image component reads hotspot and crop metadata from the Sanity image object and generates crop-aware URLs via the image URL builder. Editors can set focal points in Studio.
- **D-06:** Old `strapi-image.tsx` stays as-is during Phase 2. New `sanity-image.tsx` created alongside. Layout components switch to the new one. Clean removal in Phase 6.

**Block Renderer Transition**
- **D-07:** Big bang switch — update `COMPONENT_REGISTRY` keys from `layout.hero-section` format to `heroSection` format all at once. Change `BaseBlock` discriminator from `__component` to `_type`, add `_key` field. No dual-registry period.
- **D-08:** Update all auxiliary sets in the same phase: `NON_RENDERABLE_COMPONENTS` keys become Sanity `_type` names (e.g., `generalInquiryLabels`), `SKIP_ANIMATION` keys become Sanity `_type` names (e.g., `heroSection`). Everything stays consistent.

**Type Definitions Strategy**
- **D-09:** Keep single barrel `types/index.ts` file. Replace all Strapi-shaped types with Sanity-shaped ones. Remove `TStrapiResponse<T>` wrapper entirely — GROQ returns plain objects directly.
- **D-10:** Remove all Strapi-specific types in this phase (TStrapiResponse, TImage with Strapi shape, Strapi rich text types like BlocksContent). Clean break — nothing references them after loaders switch to GROQ.
- **D-11:** Move all block union types (TBlocks, TContactPageBlocks, TGalleryPageBlocks, etc.) into `types/index.ts`. Pages import from `@/types`. Eliminates the current circular import pattern where `types/index.ts` imports from page files.

### Claude's Discretion

- Exact GROQ query projections for each loader (field selection, nested object expansion)
- Sanity image URL builder configuration (default quality, format settings)
- Internal structure of new Sanity TypeScript types (field names follow schema, specific optional/required decisions)
- `getBlockKey` function update to use `_key` instead of `documentId`/`id`
- Whether to keep `getStrapiURL()` and `getStrapiMedia()` utilities or just leave them in the renamed files

### Deferred Ideas (OUT OF SCOPE)

None — discussion stayed within phase scope.
</user_constraints>

---

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| DATA-01 | GROQ-based data loaders replace all 8 functions in `loaders.ts` | GROQ singleton query pattern, `client.fetch<T>()` usage, `_type` projection in blocks arrays |
| DATA-02 | Block renderer `COMPONENT_REGISTRY` keys updated from `layout.hero-section` to `heroSection` | `_type` key mapping table documented, `BaseBlock` discriminator change from `__component` to `_type` |
| DATA-03 | TypeScript types rewritten to match Sanity document shapes | Sanity image type shape, Portable Text type, block key field `_key`, plain object returns from GROQ |
| DATA-04 | `generateStaticParams` rewritten from hardcoded slugs to GROQ query | GROQ `slug.current` projection pattern for collection |
| DATA-05 | All 5 content pages render correctly from Sanity data | Prop shape changes between Strapi and Sanity documented per component, error handling rewrite |
| IMG-01 | `sanity-image.tsx` replaces `strapi-image.tsx` using `@sanity/image-url` | Builder API verified against installed v1.2.0, hotspot/crop params documented |
| IMG-02 | All images uploaded to Sanity CDN with hotspot and crop | Image object shape from Sanity GROQ (asset._ref, hotspot, crop) documented |
| IMG-03 | `cdn.sanity.io` added to `next.config.ts` remote patterns | remotePatterns config snippet provided |
| IMG-04 | Sanity image transforms via URL params available | width, height, format, auto params available via builder chain |
</phase_requirements>

---

## Summary

Phase 2 replaces every Strapi-specific data layer artifact with Sanity equivalents: REST-based loaders become GROQ-based loaders using `client.fetch<T>()`, the `TStrapiResponse<T>` envelope type disappears because GROQ returns plain objects, and `TImage` (with Strapi's `url`/`alternativeText` shape) becomes Sanity's `{ asset: { _ref }, hotspot, crop, alt }` shape. All 14 layout components that currently accept Strapi-shaped props need their prop types updated to match Sanity schema field names; most field names are identical, but the image field type and the rich text field type (BlocksContent to PortableText) differ significantly.

The block renderer change is straightforward but high-impact: every registry key, the `BaseBlock` discriminator field, and the `getBlockKey` function all flip from Strapi conventions to Sanity conventions in one commit. The `COMPONENT_REGISTRY` currently maps 9 block types (plus 1 non-renderable); after this phase it maps using camelCase `_type` keys matching the Sanity schema names. The gallery component has a critical field name mismatch: the Strapi prop is `image` (singular) but the Sanity `photoGallery` schema field is `images` (plural) — the GROQ projection must alias or the gallery component prop must update.

The image pipeline requires installing `@sanity/image-url` (already present as a transitive dep at v1.2.0) and building a `SanityImage` component. The `sanity-image.tsx` component wraps the `imageUrlBuilder` from `@sanity/image-url`, initialized with the Sanity client config. Adding `cdn.sanity.io` to `next.config.ts` remotePatterns is a one-line change. The `validateApiResponse()` and `handleApiError()` utilities in `error-handler.ts` are tightly coupled to `TStrapiResponse<T>` and will need replacement with simpler null-checks since GROQ throws on error.

**Primary recommendation:** Build in this order: (1) types, (2) image component, (3) loaders, (4) block renderer, (5) update layout components, (6) update pages and layout. This order ensures each layer compiles before the next is touched.

---

## Standard Stack

### Core

| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| `next-sanity` | 9.12.3 (installed) | `createClient` factory for Next.js, App Router compatible | Official next-sanity package with built-in caching support |
| `@sanity/client` | 7.20.0 (installed) | Sanity read client used directly by loaders | Peer dep of next-sanity; `client.fetch<T>()` is the canonical GROQ call |
| `groq` | 3.99.0 (installed, via next-sanity) | Query language tag for GROQ strings | Provides `groq` template literal tag for syntax highlighting |
| `@sanity/image-url` | 1.2.0 (installed, transitive) | Build cdn.sanity.io URLs with transform params | Official Sanity image URL builder with hotspot/crop support |

### Supporting

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| `next/image` | built-in | Image component with layout optimization | Wrap with `SanityImage`, always use instead of `<img>` |
| `next/cache` (unstable_cache) | built-in | Cache GROQ results for global/menu data | Use in `(site)/layout.tsx` for global data and menu — same pattern as Strapi version |

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| `@sanity/image-url` v1.2.0 (installed) | Upgrade to v2.0.3 | v2.0.3 has breaking API changes but is not needed — v1.2.0 is fully functional. Do NOT upgrade in this phase. |
| `groq` template tag | Plain template strings | Template tag enables IDE syntax highlighting; negligible cost |

**Installation:** No additional packages needed. All required packages are already installed.

**Version verification:**
- `@sanity/image-url`: 1.2.0 (in root `node_modules`, verified 2026-03-25)
- `groq`: 3.99.0 (transitive via `next-sanity` and `sanity`)
- `next-sanity`: 9.12.3 (direct dep)

---

## Architecture Patterns

### Recommended File Changes

```
frontend/src/
├── data/
│   ├── loaders.ts              # RENAME to strapi-loaders.ts (untouched)
│   ├── data-api.ts             # RENAME to strapi-data-api.ts (untouched)
│   └── loaders.ts              # NEW — GROQ-based loaders (create fresh)
├── types/
│   └── index.ts                # REWRITE — Sanity types replace Strapi types
├── components/
│   ├── ui/
│   │   ├── strapi-image.tsx    # STAYS as-is (Phase 6 cleanup)
│   │   └── sanity-image.tsx    # NEW — Sanity image component
│   └── custom/layouts/
│       └── block-renderer.tsx  # UPDATE — registry keys, BaseBlock, getBlockKey
├── app/(site)/
│   ├── layout.tsx              # UPDATE — use new loaders, remove validateApiResponse
│   ├── page.tsx                # UPDATE — use new loaders, remove validateApiResponse
│   ├── contact/page.tsx        # UPDATE — use new loaders
│   ├── gallery/page.tsx        # UPDATE — use new loaders
│   ├── training/page.tsx       # UPDATE — use new loaders
│   └── training-programs/[slug]/page.tsx  # UPDATE — GROQ generateStaticParams
└── next.config.ts              # UPDATE — add cdn.sanity.io remote pattern
```

### Pattern 1: GROQ Singleton Document Fetch

**What:** Fetch a singleton document (homePage, contactPage, etc.) by `_type` and `_id` using the documentId that matches the type name (per Phase 1 D-11 decision: singleton documentId matches type name).

**When to use:** Every singleton page loader.

```typescript
// Source: next-sanity conventions + Phase 1 CONTEXT.md D-11
import { client } from "@/sanity/lib/client";
import { groq } from "next-sanity";

async function getHomePageData(): Promise<THomePage | null> {
  const query = groq`*[_type == "homePage"][0]{
    title,
    description,
    blocks[]{
      _type,
      _key,
      ...
    }
  }`;
  return client.fetch<THomePage | null>(query);
}
```

**Key insight:** Singletons are fetched with `[0]` at the end — the dataset has exactly one document of each singleton type. No `_id` filter needed because only one exists.

### Pattern 2: GROQ Block Array Projection with _type Expansion

**What:** Expand typed block arrays from Sanity documents, projecting only the needed fields.

**When to use:** All loaders that fetch documents with `blocks` arrays.

```typescript
// Source: Sanity GROQ documentation + schema inspection
const query = groq`*[_type == "homePage"][0]{
  title,
  description,
  blocks[]{
    _type,
    _key,
    // heroSection fields
    heading,
    subHeading,
    description,
    onHomepage,
    image{
      asset,
      hotspot,
      crop,
      alt
    },
    link[]{
      href,
      label,
      isExternal
    },
    // featuresSection fields
    features[]{
      _key,
      heading,
      subHeading,
      icon
    }
  }
}`;
```

**Important:** Unlike Strapi's `populate.on` pattern, GROQ projects ALL block types simultaneously — every field mentioned in the projection is returned for blocks that have it, null for blocks that don't. This means a flat projection covering all block types in a page works correctly.

### Pattern 3: GROQ Collection Query with Slug Filter

**What:** Fetch a single `trainingProgram` document by slug.

**When to use:** `getTrainingProgramBySlug(slug)` loader, and `generateStaticParams`.

```typescript
// Source: Sanity GROQ documentation
async function getTrainingProgramBySlug(slug: string): Promise<TTrainingProgram | null> {
  const query = groq`*[_type == "trainingProgram" && slug.current == $slug][0]{
    _id,
    title,
    description,
    slug,
    blocks[]{
      _type,
      _key,
      heading,
      subHeading,
      // ... all block fields
    }
  }`;
  return client.fetch<TTrainingProgram | null>(query, { slug });
}

async function getAllTrainingProgramSlugs(): Promise<Array<{ slug: string }>> {
  const query = groq`*[_type == "trainingProgram"]{ "slug": slug.current }`;
  return client.fetch<Array<{ slug: string }>>(query);
}
```

**For generateStaticParams:**
```typescript
export async function generateStaticParams() {
  const programs = await loaders.getAllTrainingProgramSlugs();
  return programs.map(p => ({ slug: p.slug }));
}
```

### Pattern 4: Sanity Image Component

**What:** New `SanityImage` component using `@sanity/image-url` v1.2.0 builder.

**When to use:** Replace every `<StrapiImage>` call in layout components.

```typescript
// Source: @sanity/image-url v1.2.0 API inspection (installed)
import Image from "next/image";
import imageUrlBuilder from "@sanity/image-url";
import { client } from "@/sanity/lib/client";

const builder = imageUrlBuilder(client);

interface SanityImageAsset {
  asset: { _ref: string };
  hotspot?: { x: number; y: number; width: number; height: number };
  crop?: { top: number; bottom: number; left: number; right: number };
  alt?: string;
}

interface ISanityImageProps {
  image: SanityImageAsset;
  alt?: string;
  width?: number;
  height?: number;
  className?: string;
  fill?: boolean;
  priority?: boolean;
}

export function SanityImage({ image, alt, className, priority, ...rest }: ISanityImageProps) {
  if (!image?.asset?._ref) return null;

  const url = builder
    .image(image)
    .auto("format")
    .fit("max")
    .url();

  return (
    <Image
      src={url}
      alt={alt ?? image.alt ?? "No alternative text provided"}
      className={className}
      priority={priority}
      {...rest}
    />
  );
}
```

**Hotspot/crop:** The builder automatically reads `hotspot` and `crop` from the image object when present — no additional API call needed.

### Pattern 5: Updated BaseBlock Interface

**What:** Updated discriminator and key fields for Sanity.

```typescript
// Replaces current BaseBlock in block-renderer.tsx
interface BaseBlock {
  _type: string;   // was __component
  _key?: string;   // was id/documentId
}

// Updated getBlockKey
function getBlockKey(block: BaseBlock, index: number): string {
  if (block._key) return `${block._type}-${block._key}`;
  return `${block._type}-${index}`;
}
```

### Pattern 6: Updated COMPONENT_REGISTRY

**What:** New registry keys using Sanity camelCase `_type` names.

```typescript
const COMPONENT_REGISTRY = {
  "heroSection": HeroSection,
  "featuresSection": FeaturesSection,
  "ctaFeaturesSection": CtaFeaturesSection,
  "imageWithText": ImageWithText,
  "infoSection": InfoSection,
  "photoGallery": Gallery,
  "schedule": Schedule,
  "contactInfo": ContactInfo,
  "contactFormLabels": ContactFormLabels,
} as const;

const NON_RENDERABLE_COMPONENTS = new Set([
  "generalInquiryLabels",  // was "layout.general-inquiry-labels"
]);

const SKIP_ANIMATION = new Set([
  "heroSection",  // was "layout.hero-section"
]);
```

### Pattern 7: Error Handling Without TStrapiResponse

**What:** GROQ `client.fetch()` throws on network/auth errors and returns `null` when a document is not found (with `[0]` on empty array). Replace `validateApiResponse()` pattern.

```typescript
// In pages: GROQ loaders return T | null directly
const data = await loaders.getHomePageData();
if (!data) notFound();

// No validateApiResponse() call needed
// No success/error field checks
```

**In `(site)/layout.tsx`:** The `unstable_cache` wrappers remain but the inner function changes from `validateApiResponse(response)` to a null check.

### Pattern 8: Global Data Loader (for layout.tsx)

**What:** Global settings and menu require separate loaders cached with `unstable_cache`.

```typescript
// globalSettings is a singleton
async function getGlobalData(): Promise<TGlobalSettings | null> {
  const query = groq`*[_type == "globalSettings"][0]{
    title,
    description,
    header{
      logoText{ href, label, isExternal },
      ctaButton[]{ href, label, isExternal }
    },
    footer{
      logoText{ href, label, isExternal },
      text,
      socialLink[]{ href, label, isExternal }
    }
  }`;
  return client.fetch<TGlobalSettings | null>(query);
}

// mainMenu is a singleton
async function getMainMenuData(): Promise<TMainMenu | null> {
  const query = groq`*[_type == "mainMenu"][0]{
    items[]{
      _type,
      _key,
      title,
      url,
      sections[]{
        _key,
        heading,
        links[]{
          _key,
          name,
          url,
          description
        }
      }
    }
  }`;
  return client.fetch<TMainMenu | null>(query);
}
```

### Anti-Patterns to Avoid

- **Importing `TStrapiResponse<T>` in new files:** GROQ returns plain objects. Any `TStrapiResponse` import in new files is wrong.
- **Using `api.get()` from `data-api.ts` in new loaders:** The new loaders use `client.fetch()` directly. The old `api` object is only in the renamed `strapi-data-api.ts`.
- **Calling `qs.stringify()` in new loaders:** qs is for building Strapi URL query strings. GROQ uses parameterized queries (`client.fetch(query, { slug })`).
- **Projecting all fields with `...` spread alone:** Sanity objects (images, linked documents) need explicit expansion. `image` returns just the asset reference unless you project `image{ asset, hotspot, crop, alt }`.
- **Using `client.fetch()` in Client Components:** GROQ fetches must be in Server Components or server-only modules. The Sanity read client has `useCdn: true` — safe for server-side read fetching.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Image URL generation with hotspot/crop | Custom URL string builder | `@sanity/image-url` builder | Handles Sanity asset ID format, crop math, CDN URL structure — edge cases are non-trivial |
| GROQ query sanitization | Custom string escaping | Parameterized queries: `client.fetch(query, { slug })` | Prevents injection; handles Sanity's parameter encoding |
| Singleton document fetch pattern | Custom document ID lookup | `*[_type == "homePage"][0]` | This is the Sanity canonical singleton pattern |
| Image dimensions inference | Reading Sanity image metadata separately | `builder.image(img).width(N).height(N)` | Builder knows Sanity CDN transform API |

**Key insight:** The Sanity CDN URL format (`cdn.sanity.io/images/{projectId}/{dataset}/{assetId}`) with transform params (`?w=800&h=600&auto=format&fit=crop&fp-x=0.5&fp-y=0.5`) is not documentation-stable. Always use `@sanity/image-url` to generate these URLs.

---

## Field Name Change Map (Strapi to Sanity)

This is the critical lookup table for updating prop types and layout components.

### Sanity Image Object Shape (replaces Strapi TImage)

```
Strapi TImage:                    Sanity image field:
{                                 {
  id: number,                       asset: { _ref: string },  // "image-{id}-{w}x{h}-{format}"
  documentId: string,               hotspot?: {               // optional, set by editor
  url: string,          ──►           x: number,
  alternativeText: string             y: number,
}                                     width: number,
                                      height: number
                                    },
                                    crop?: {
                                      top: number,
                                      bottom: number,
                                      left: number,
                                      right: number
                                    },
                                    alt?: string              // inline field on image type
                                  }
```

### Block Discriminator Field Change

| Field | Strapi | Sanity |
|-------|--------|--------|
| Type discriminator | `__component: "layout.hero-section"` | `_type: "heroSection"` |
| Unique key | `id: number` or `documentId: string` | `_key: string` (nanoid, assigned by Sanity) |

### Per-Block Field Name Comparison

| Block | Strapi field | Sanity field | Changed? |
|-------|-------------|--------------|----------|
| `heroSection` | `image.url`, `image.alternativeText` | `image.asset._ref`, `image.alt` | YES — image shape |
| `heroSection` | `link: TLink[]` | `link: TLink[]` | NO — same field name |
| `featuresSection` | `features[].id` | `features[]._key` | YES — key field |
| `photoGallery` | `image: TImage[]` | `images: SanityImage[]` | YES — field name pluralized |
| `imageWithText` | `perks: BlocksContent` | `perks: PortableTextBlock[]` | YES — rich text shape |
| `ctaFeaturesSection` | `features[].features: BlocksContent` | `features[].features: PortableTextBlock[]` | YES — rich text shape |
| `infoSection` | `info: BlocksContent` | `info: PortableTextBlock[]` | YES — rich text shape |
| `imageWithText` | `imageLocation` field | Not in Sanity schema | REMOVED (not in schema) |
| `schedule` | `hours[].id` | `hours[]._key` | YES — key field |
| `contactInfo` | `contact[].id` | `contact[]._key` | YES — key field |
| `mainMenu` | `MainMenuItems` | `items` | YES — renamed in D-Phase 1 |
| `mainMenu items` | `__component: "menu.menu-link"` | `_type: "menuDirectLink"` | YES |
| `mainMenu items` | `__component: "menu.dropdown"` | `_type: "menuDropdown"` | YES |

### Critical Issues Requiring Code Changes in Layout Components

**gallery.tsx** — The component accesses `data.image` (Strapi field name) but the Sanity schema field is `images` (plural). Either: (a) GROQ projection aliases it: `"image": images[]{...}`, or (b) update the component prop to use `images`. Option (b) is cleaner (truthful field name). The `IGalleryProps` interface and the destructuring in the component body both need updating.

**main-menu.tsx** — The component currently discriminates on `__component === "menu.menu-link"` and `__component === "menu.dropdown"`. After migration these become `_type === "menuDirectLink"` and `_type === "menuDropdown"`. The `mainMenuRenderer` switch statement and the `IMainMenuItems` union type both need updating.

**hero-section.tsx** — Accesses `image.url` and `image.alternativeText`. Must change to generate URL via `SanityImage` component passing the full image object, and `image.alt` for alt text.

**CtaFeaturesSection and components using BlocksContent** — Currently renders `<BlockRenderer content={item.features} />` using the Strapi-format `ui/block-renderer.tsx` rich text renderer. After this phase, those fields contain Portable Text blocks, not Strapi Block Editor JSON. Phase 3 builds the Portable Text renderer — in Phase 2, these fields should still be typed correctly (as `PortableTextBlock[]`) but may render as a placeholder or nothing until Phase 3. The planner must decide whether to stub the rich text render or leave it blank for now.

**header.tsx and footer.tsx** — Currently receive `data.header` and `data.footer` from `TGlobal`. Field shapes are identical (logoText, ctaButton, socialLink) — only the inner `TImage` fields on any nested images change. The `header` and `footer` objects have no image fields, so these components may not need prop changes beyond the link shape.

---

## Common Pitfalls

### Pitfall 1: Gallery Field Name Mismatch

**What goes wrong:** The gallery component uses `data.image` (Strapi: `image` array) but the Sanity `photoGallery` schema field is `images`. A GROQ query that returns `images` will populate `undefined` in the current component.

**Why it happens:** Strapi used `image` (singular) for the array; the Sanity schema was named `images` (plural) to better reflect the array nature.

**How to avoid:** Decide in the PLAN whether to: (a) update `IGalleryProps` to use `images` field and update the component, or (b) GROQ-alias the field back to `image` with `"image": images[]{...}`. Option (a) is preferred — truthful naming.

**Warning signs:** Gallery section renders nothing; `data.image is undefined` in dev console.

### Pitfall 2: Rich Text Fields Type Collision

**What goes wrong:** `imageWithText.perks`, `infoSection.info`, and `ctaFeature.features` currently type as `BlocksContent` (Strapi Block Editor JSON). After GROQ fetch these fields contain Portable Text (`PortableTextBlock[]`). If the types don't change, TypeScript won't error but the runtime render of `<BlockRenderer content={...}>` (which expects Strapi format) will break or render nothing.

**Why it happens:** Phase 3 builds the Portable Text renderer. Phase 2 must update the types to `PortableTextBlock[]` even though the renderer doesn't exist yet.

**How to avoid:** In Phase 2, type the rich text fields as `PortableTextBlock[]` from `@portabletext/types` (or a local type alias) and either skip rendering them or render a placeholder comment. Do NOT leave them typed as `BlocksContent`.

**Warning signs:** `ui/block-renderer.tsx` (Strapi rich text) crashes or renders garbage content.

### Pitfall 3: imageUrlBuilder Initialization Per-Request

**What goes wrong:** If `imageUrlBuilder(client)` is called inside the component function body, a new builder instance is created on every render.

**Why it happens:** The builder is instantiated from the client config. Calling it inside render is wasteful.

**How to avoid:** Initialize the builder at module level, outside the component:
```typescript
const builder = imageUrlBuilder(client);  // module-level constant
```

**Warning signs:** No functional problem, but performance degradation at scale.

### Pitfall 4: Missing cdn.sanity.io in remotePatterns

**What goes wrong:** Next.js blocks external images by default. Without adding `cdn.sanity.io` to `next.config.ts` remotePatterns, every Sanity image will fail to load with a 400 error.

**Why it happens:** `next/image` requires explicit opt-in for external hostnames.

**How to avoid:** Add to `next.config.ts` BEFORE layout components switch to `SanityImage`:
```typescript
{ protocol: "https", hostname: "cdn.sanity.io", pathname: "/images/**" }
```

**Warning signs:** Images show broken placeholder; Next.js logs "hostname not configured in images.remotePatterns".

### Pitfall 5: menu.tsx Discriminator Still Checking Strapi __component

**What goes wrong:** `mainMenuRenderer()` in `main-menu.tsx` switches on `MainMenuItem.__component`. After migration, items have `_type` not `__component`. All menu items render `null` — navigation disappears.

**Why it happens:** The file is not in the block-renderer.tsx registry update path and is easy to miss.

**How to avoid:** Update `main-menu.tsx` in the same plan wave as block-renderer: change switch to `_type`, update `IMainMenuItems` union, update field access from `MainMenuItems` to `items`.

**Warning signs:** Header renders with no navigation links.

### Pitfall 6: Stale Strapi Imports in Renamed Files

**What goes wrong:** After renaming `loaders.ts` to `strapi-loaders.ts`, the new `loaders.ts` must have no imports from `qs`, `data-api`, or `getStrapiURL`. If any leftover Strapi import sneaks in (e.g., via copy-paste from the old file), TypeScript may not catch it immediately.

**Why it happens:** Copy-paste from old loaders to new ones.

**How to avoid:** Start the new `loaders.ts` from scratch with only `client` and `groq` imports. Verify with `import check`: the new file must not reference `qs`, `api`, `getStrapiURL`, or `TStrapiResponse`.

### Pitfall 7: Error Handling with GROQ — null vs throw

**What goes wrong:** `client.fetch()` returns `null` when `[0]` is applied to an empty result (document not found). It throws on auth/network errors. `validateApiResponse()` from `error-handler.ts` expects `TStrapiResponse<T>` — calling it on a plain `T | null` result will cause a TypeScript error and runtime failure.

**Why it happens:** Error handling is coupled to the Strapi response envelope.

**How to avoid:** Pages use direct null checks: `if (!data) notFound()` or `if (!data) throw new Error(...)`. The `error-handler.ts` file stays but is not used by new loaders. The Strapi-era pattern of `validateApiResponse(response, "page name")` is retired for all new code.

---

## Code Examples

### Verified Pattern: GROQ Singleton with Full Blocks Projection (Homepage)

```typescript
// Source: Sanity schema inspection (home-page.ts, hero-section.ts, features-section.ts)
const getHomePageData = async (): Promise<THomePage | null> => {
  const query = groq`*[_type == "homePage"][0]{
    title,
    description,
    blocks[]{
      _type,
      _key,
      // heroSection
      heading,
      subHeading,
      description,
      onHomepage,
      image{
        asset,
        hotspot,
        crop,
        alt
      },
      link[]{
        _key,
        href,
        label,
        isExternal
      },
      // featuresSection
      features[]{
        _key,
        heading,
        subHeading,
        icon
      },
      title
    }
  }`;
  return client.fetch<THomePage | null>(query);
};
```

### Verified Pattern: Training Program Collection by Slug

```typescript
// Source: Sanity schema inspection (training-program.ts)
const getTrainingProgramBySlug = async (slug: string): Promise<TTrainingProgram | null> => {
  const query = groq`*[_type == "trainingProgram" && slug.current == $slug][0]{
    _id,
    title,
    description,
    "slug": slug.current,
    blocks[]{
      _type,
      _key,
      // heroSection fields
      heading,
      subHeading,
      description,
      onHomepage,
      image{ asset, hotspot, crop, alt },
      link[]{ _key, href, label, isExternal },
      // infoSection fields
      info,
      // contactFormLabels fields
      name,
      email,
      phone,
      instagram,
      location,
      interest,
      experience,
      clients,
      info
    }
  }`;
  return client.fetch<TTrainingProgram | null>(query, { slug });
};
```

### Verified Pattern: generateStaticParams from GROQ

```typescript
// Source: DATA-04 requirement + Sanity slug.current pattern
export async function generateStaticParams() {
  const programs = await loaders.getAllTrainingProgramSlugs();
  return programs.map(p => ({ slug: p.slug }));
}

// In loaders.ts:
const getAllTrainingProgramSlugs = async (): Promise<Array<{ slug: string }>> => {
  const query = groq`*[_type == "trainingProgram"]{ "slug": slug.current }`;
  return client.fetch<Array<{ slug: string }>>(query);
};
```

### Verified Pattern: Sanity Image Component (v1.2.0 API)

```typescript
// Source: @sanity/image-url v1.2.0 installed at /node_modules/@sanity/image-url
import Image from "next/image";
import imageUrlBuilder from "@sanity/image-url";
import { client } from "@/sanity/lib/client";
import type { SanityImageSource } from "@sanity/image-url/lib/types/types";

const builder = imageUrlBuilder(client);

function urlFor(source: SanityImageSource) {
  return builder.image(source);
}

interface TSanityImage {
  asset: { _ref: string };
  hotspot?: { x: number; y: number; width: number; height: number };
  crop?: { top: number; bottom: number; left: number; right: number };
  alt?: string;
}

interface ISanityImageProps {
  image: TSanityImage;
  alt?: string;
  width?: number;
  height?: number;
  className?: string;
  fill?: boolean;
  priority?: boolean;
}

export function SanityImage({ image, alt, className, priority, width, height, fill }: ISanityImageProps) {
  if (!image?.asset?._ref) return null;

  const url = urlFor(image).auto("format").fit("max").url();

  return (
    <Image
      src={url}
      alt={alt ?? image.alt ?? ""}
      className={className}
      priority={priority}
      {...(fill ? { fill } : { width: width ?? 800, height: height ?? 600 })}
    />
  );
}
```

### Verified Pattern: next.config.ts Remote Patterns Update

```typescript
// Source: next.config.ts inspection
images: {
  remotePatterns: [
    // ADD:
    {
      protocol: "https",
      hostname: "cdn.sanity.io",
      pathname: "/images/**",
    },
    // KEEP existing Strapi patterns until Phase 6:
    {
      protocol: "http",
      hostname: "localhost",
      port: "1337",
      pathname: "/uploads/**/*",
    },
    {
      protocol: "https",
      hostname: "cheerful-prosperity-b55f073006.media.strapiapp.com",
      pathname: "/**",
    },
    {
      protocol: "https",
      hostname: "*.public.blob.vercel-storage.com",
      pathname: "/**",
    },
  ],
},
```

### Verified Pattern: Core Sanity Types for types/index.ts

```typescript
// Sanity image object type (replaces TImage)
export interface TSanityImage {
  asset: { _ref: string; _type: "reference" };
  hotspot?: { x: number; y: number; width: number; height: number };
  crop?: { top: number; bottom: number; left: number; right: number };
  alt?: string;
}

// Sanity link type (same field names as Strapi TLink — no prop changes needed)
export interface TLink {
  _key?: string;
  href: string;
  label: string;
  isExternal?: boolean;
}

// Portable Text block type (for infoSection.info, imageWithText.perks, ctaFeature.features)
// Use @portabletext/types if available, or define inline:
export type TPortableTextBlock = {
  _type: "block";
  _key: string;
  style: string;
  children: Array<{ _type: "span"; _key: string; text: string; marks: string[] }>;
  markDefs: Array<{ _type: string; _key: string; [key: string]: unknown }>;
};

// BaseBlock for block renderer
export interface TSanityBlock {
  _type: string;
  _key?: string;
}
```

---

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Strapi `populate.on` syntax for typed zone queries | GROQ flat projection covering all block types | Phase 2 | Simpler — no per-component populate needed |
| `TStrapiResponse<T>` envelope for all API calls | Plain `T \| null` from GROQ | Phase 2 | Remove `success`/`error` field checks everywhere |
| `qs.stringify()` for query building | GROQ template literals with `$param` variables | Phase 2 | Cleaner, type-safe parameterization |
| `__component` discriminator in block arrays | `_type` discriminator | Phase 2 | Sanity's built-in type field |

**Deprecated/outdated in this phase:**
- `TStrapiResponse<T>`: retired — never import in new files
- `validateApiResponse()`: retired for new pages — direct null check instead
- `getStrapiURL()` / `getStrapiMedia()`: stay in renamed `strapi-data-api.ts` only
- `qs` package: stays in `strapi-loaders.ts` only

---

## Open Questions

1. **Rich text fields in Phase 2 blocks**
   - What we know: `imageWithText.perks`, `infoSection.info`, `ctaFeature.features` are Portable Text in Sanity. The Phase 3 renderer doesn't exist yet.
   - What's unclear: Should Phase 2 render a placeholder for these fields, or leave them as unrendered data?
   - Recommendation: Type them as `TPortableTextBlock[]` in Phase 2 types, but stub the render call with `null` or a `<p>[rich text]</p>` placeholder. Phase 3 replaces the stub. This keeps TypeScript strict without blocking Phase 2 completion.

2. **Contact page: blocks vs. direct page fields**
   - What we know: The contact page currently uses `validateApiResponse` + a `blocks.find()` pattern to extract schedule, contact-info, and general-inquiry-labels from the blocks array. The Sanity `contactPage` schema has `title`, `subTitle`, `description` as direct fields, plus a `blocks` array.
   - What's unclear: The `GeneralInquiryLayout` component receives page-level fields (`title`, `subTitle`, `description`) alongside block data — the GROQ loader must return all of these.
   - Recommendation: The GROQ projection must include both the direct fields AND the blocks array projection. Document this clearly in the plan.

3. **imageLocation field on imageWithText**
   - What we know: The Strapi `TImageWithText` type has an `imageLocation` field. The Sanity `imageWithText` schema has `orientation` but no `imageLocation`.
   - What's unclear: Is `imageLocation` actually used in the component render, or was it unused?
   - Recommendation: Check `image-with-text.tsx` component — if it references `imageLocation`, that prop must be removed from the type and the component updated to use `orientation` instead.

---

## Environment Availability

| Dependency | Required By | Available | Version | Fallback |
|------------|------------|-----------|---------|----------|
| `next-sanity` | All GROQ loaders | Yes | 9.12.3 | — |
| `@sanity/client` | `client.fetch()` | Yes | 7.20.0 | — |
| `@sanity/image-url` | `sanity-image.tsx` | Yes | 1.2.0 (in root node_modules) | — |
| `groq` tag | GROQ query strings | Yes | 3.99.0 (via next-sanity) | Plain template strings |
| Sanity project credentials | `client.fetch()` at runtime | Assumed yes | — | Phase fails if env vars missing |

**Missing dependencies with no fallback:** None.

**Note on @sanity/image-url location:** The package is at `/node_modules/@sanity/image-url` (root workspace), not `/frontend/node_modules/`. npm workspaces hoisting puts it at root. Import resolution works correctly from the frontend workspace — no action needed.

---

## Validation Architecture

### Test Framework

| Property | Value |
|----------|-------|
| Framework | Playwright 1.58.2 |
| Config file | `frontend/playwright.config.ts` |
| Quick run command | `cd frontend && npx playwright test --project=chromium homepage.spec.ts` |
| Full suite command | `cd frontend && npx playwright test` |

### Phase Requirements to Test Map

| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| DATA-01 | All pages load without errors from Sanity | E2E smoke | `npx playwright test --project=chromium` | Yes (all spec files) |
| DATA-02 | All 14 block types render — no blank/unrecognized blocks | E2E visual | `npx playwright test homepage.spec.ts gallery.spec.ts training.spec.ts` | Yes |
| DATA-03 | TypeScript compiles with strict mode | Build check | `cd frontend && npx tsc --noEmit` | n/a — tsc command |
| DATA-04 | `/training-programs/[slug]` resolves correctly per slug | E2E smoke | `npx playwright test --project=chromium training.spec.ts` | Yes |
| DATA-05 | All 5 pages render content from Sanity | E2E smoke | `npx playwright test --project=chromium` | Yes |
| IMG-01 | `SanityImage` renders without broken placeholder | E2E visual | `npx playwright test homepage.spec.ts` | Yes (partial) |
| IMG-02 | Images load from cdn.sanity.io | E2E network | Manual observation or playwright network intercept | Yes (partial) |
| IMG-03 | cdn.sanity.io remote pattern active | Build/runtime | `cd frontend && npx next build 2>&1 \| grep -i "hostname"` | n/a — build check |
| IMG-04 | Image transforms work (width/height/format) | Manual | Check browser devtools Network tab | Manual |

### Sampling Rate

- **Per task commit:** `cd frontend && npx tsc --noEmit` (TypeScript check, < 15 seconds)
- **Per wave merge:** `cd frontend && npx playwright test --project=chromium` (Chromium only, faster)
- **Phase gate:** Full suite: `cd frontend && npx playwright test` before `/gsd:verify-work`

### Wave 0 Gaps

- The existing Playwright tests use `setupApiMocks(page)` which mocks Strapi API calls. After migration these mocks will not intercept Sanity GROQ calls. The mock utility needs updating or disabling for Phase 2 tests to hit the real Sanity staging dataset.
- If no Sanity staging data exists yet, all E2E tests will fail with empty pages. The plan must include a note that E2E tests require staged data.

---

## Sources

### Primary (HIGH confidence)

- Schema files at `frontend/src/sanity/schemas/` — inspected all 10 document schemas and 22 object schemas directly — highest confidence source for field names and types
- Installed packages: `@sanity/image-url@1.2.0`, `next-sanity@9.12.3`, `groq@3.99.0`, `@sanity/client@7.20.0` — version-confirmed via `npm list` and `package.json` inspection
- Phase 1 decisions in `02-CONTEXT.md` — locked decisions from user discussion
- Existing source files: `loaders.ts`, `data-api.ts`, `types/index.ts`, `block-renderer.tsx`, `strapi-image.tsx`, `next.config.ts`, all layout components — read directly

### Secondary (MEDIUM confidence)

- `@sanity/image-url` builder API — inspected from `node_modules/@sanity/image-url/lib/node/builder.js` at installed version
- GROQ singleton pattern (`*[_type == "X"][0]`) — standard Sanity pattern documented across official sources, aligned with Phase 1 D-11 decisions

### Tertiary (LOW confidence)

- None

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — all packages verified installed, versions confirmed
- Architecture patterns: HIGH — derived directly from schema files and existing source code
- Field name change map: HIGH — cross-referenced schema definitions against component prop interfaces
- Pitfalls: HIGH — derived from concrete code inspection (gallery field name, discriminator fields, error handling coupling)
- Rich text in Phase 2: MEDIUM — Phase 3 dependency creates uncertainty about how to stub Portable Text renders

**Research date:** 2026-03-25
**Valid until:** 2026-04-25 (stable ecosystem; Sanity packages update frequently but installed versions are locked)
