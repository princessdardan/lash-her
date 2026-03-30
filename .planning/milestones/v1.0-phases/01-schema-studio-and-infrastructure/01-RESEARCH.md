# Phase 1: Schema, Studio, and Infrastructure - Research

**Researched:** 2026-03-24
**Domain:** Sanity CMS — schema definition, Studio embedding, client configuration
**Confidence:** HIGH

---

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

- **D-01:** Sidebar organized into 3 sections: **Pages** (all 7 singletons), **Content** (training programs collection), **Submissions** (general inquiries + contact forms)
- **D-02:** Singletons open directly when clicked — no intermediate document list view. Use Sanity's `documentListItem().child()` pattern.
- **D-03:** Friendly document titles in sidebar: "Homepage", "Contact Page", "Gallery", "Training", "Training Programs Overview", "Global Settings", "Navigation Menu"
- **D-04:** Block array items in Studio show compact type list with key field preview (e.g., "Hero Section — Welcome to Lash Her"), not expanded field previews
- **D-05:** All Sanity code lives under `frontend/src/sanity/` — schemas, client config, Studio structure, and `sanity.config.ts`
- **D-06:** Schemas grouped into `schemas/documents/` (10 document types) and `schemas/objects/` (14 layout block types + 6 reusable sub-objects), with a barrel `schemas/index.ts`
- **D-07:** Schema files use kebab-case filenames matching the existing frontend convention (e.g., `hero-section.ts`, `home-page.ts`)
- **D-08:** Each page document defines its own allowed block types in its `blocks` array field — per-page type restrictions, not a shared global set.
- **D-09:** Block array field named `blocks` on all page documents — same as Strapi.
- **D-10:** Mirror Strapi's structure 1:1 — `trainingPage` and `trainingProgramsPage` remain separate singleton documents
- **D-11:** Skip the unused `section` collection type — no Sanity schema for it
- **D-12:** `contact-form` (layout) becomes `contactFormLabels`; `general-inquiry-labels` becomes `generalInquiryLabels`
- **D-13:** Rich text fields defined as Portable Text (`block` type) from day one, even though the renderer is built in Phase 3

### Claude's Discretion

- Icon choices for sidebar document types
- Exact Portable Text block configuration (which decorators, annotations, styles to enable)
- Sanity `preview` configuration details for each schema type
- Internal structure of reusable sub-objects (field ordering, validation rules)

### Deferred Ideas (OUT OF SCOPE)

None — discussion stayed within phase scope.
</user_constraints>

---

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| SCHEMA-01 | Sanity schema for all 7 singleton documents using singleton pattern | Schema definition patterns, singleton config, `defineConfig` templates/actions |
| SCHEMA-02 | Sanity schema for collection types (contactForm, generalInquiry, trainingProgram) | Collection type schema patterns, no `draftAndPublish` for submissions |
| SCHEMA-03 | 14 layout object types mirroring Strapi layout components | Object type definition, block array, image fields, Portable Text |
| SCHEMA-04 | 6 reusable sub-object types (link, menuLink, feature, ctaFeature, contact, hours) | Object type reuse via `type` reference in `of` arrays |
| SCHEMA-05 | Navigation menu schema supports polymorphic items | Array `of` multiple types with `_type` discriminator, `menu.dropdown` schema |
| INFRA-01 | Sanity Studio embedded at `/studio` via `next-sanity` `NextStudio` component | `next-sanity/studio` NextStudio, `[[...tool]]` route, `force-static` |
| INFRA-02 | Two Sanity projects created — staging and production | Manual step via `sanity.io/manage` or `npx sanity projects create` |
| INFRA-03 | Read client configured (`useCdn: true`) for production content fetching | `createClient` from `next-sanity` with `useCdn: true` |
| INFRA-04 | Write client configured (server-only, `SANITY_WRITE_TOKEN`) | `createClient` with token, `server-only` package import |
</phase_requirements>

---

## Summary

Phase 1 establishes the entire Sanity foundation: schema definitions, embedded Studio, and client configuration. No frontend changes are made — this phase produces only the Sanity side of the integration that later phases build on.

The Sanity schema mirrors the existing Strapi content model. 7 singleton document types map directly to Strapi single types; 3 collection types map to Strapi collection types; 14 layout object types map to Strapi `layout.*` components; and 6 sub-object types map to Strapi `components.*` components. The one intentional deviation from Strapi is renaming `layout.contact-form` to `contactFormLabels` (D-12) to avoid collision with the `contactForm` submission document.

The Studio embeds in Next.js at `/studio` using `next-sanity`'s `NextStudio` component with a catch-all route. Structure is configured with 3 sidebar sections (Pages, Content, Submissions), singletons open directly without intermediate document lists, and the schema prevents duplicate singleton creation via `schema.templates` filtering and `document.actions` restriction.

**Primary recommendation:** Define all schemas using `defineType`/`defineField`/`defineArrayMember` for type safety; configure the structure builder as a flat sidebar with dividers; place the write client in a `server-only` module; use `NEXT_PUBLIC_` prefix only for non-secret env vars (projectId, dataset, apiVersion).

---

## Project Constraints (from CLAUDE.md)

- **Package manager:** npm (no yarn/pnpm)
- **No `useMemo`/`useCallback`/`React.memo`** — React Compiler handles memoization
- **TypeScript strict:** explicit types on all function signatures, no `any`, strict null checks
- **`interface` for object shapes**, `type` for unions/intersections
- **Named exports** preferred; default exports only for Next.js pages/layouts
- **`const` by default**, `let` only when reassignment needed
- **Kebab-case files, PascalCase exports:** `hero-section.ts` exports `heroSection` (as a `const`, not a class)
- **`@/` maps to `frontend/src/`** — Sanity imports use `@/sanity/...`
- **Conventional Commits:** `feat:`, `chore:`, `refactor:` prefixes
- **GSD workflow enforcement:** use `/gsd:execute-phase` for all file changes

---

## Standard Stack

### Core
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| `sanity` | 5.18.0 | Schema definition, Studio runtime | Core Sanity package — all `defineType`, `defineConfig`, structure builder |
| `next-sanity` | 12.1.6 | Next.js integration — `NextStudio`, `createClient`, `groq` | Official Sanity toolkit for Next.js; bundles client + Studio wiring |
| `@sanity/client` | 7.20.0 | Sanity HTTP client (read/write) | Used by `next-sanity` internally; explicit install for write client |
| `server-only` | (latest) | Prevents server modules from importing in client bundles | Next.js App Router convention for write client protection |

### Supporting (installed in later phases but schema-relevant now)
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| `@portabletext/react` | 6.0.3 | Render Portable Text in React | Phase 3 — but the PT schema defined here feeds it |
| `@sanity/image-url` | 2.0.3 | Build CDN URLs from Sanity image references | Phase 2 — but image fields defined here |

### Alternatives Considered
| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| `next-sanity createClient` | `@sanity/client createClient` directly | `next-sanity` wraps the client with Next.js-specific cache tags support; use `next-sanity` |
| Two separate projects (INFRA-02) | Two datasets in one project | Locked decision; but see Open Questions — free tier now allows 2 datasets per project |

**Installation (in `frontend/` workspace):**
```bash
cd /Users/dardan/Documents/lash-her/frontend
npm install sanity next-sanity @sanity/client server-only
```

**Version verification (confirmed 2026-03-24):**
```
sanity:          5.18.0
next-sanity:     12.1.6
@sanity/client:  7.20.0
```

---

## Architecture Patterns

### Recommended Project Structure

```
frontend/src/sanity/
├── sanity.config.ts          # defineConfig — Studio entry point ('use client')
├── env.ts                    # NEXT_PUBLIC_ env var exports (projectId, dataset, apiVersion)
├── lib/
│   ├── client.ts             # Read client (useCdn: true)
│   └── write-client.ts       # Write client (server-only, SANITY_WRITE_TOKEN)
├── structure/
│   └── index.ts              # StructureResolver — 3-section sidebar
└── schemas/
    ├── index.ts              # Barrel: exports schemaTypes array
    ├── documents/
    │   ├── home-page.ts
    │   ├── contact-page.ts
    │   ├── gallery-page.ts
    │   ├── training-page.ts
    │   ├── training-programs-page.ts
    │   ├── global-settings.ts
    │   ├── main-menu.ts
    │   ├── training-program.ts
    │   ├── contact-form.ts
    │   └── general-inquiry.ts
    └── objects/
        ├── layout/
        │   ├── hero-section.ts
        │   ├── features-section.ts
        │   ├── cta-features-section.ts
        │   ├── cta-section-image.ts
        │   ├── cta-section-video.ts
        │   ├── image-with-text.ts
        │   ├── info-section.ts
        │   ├── photo-gallery.ts
        │   ├── schedule.ts
        │   ├── contact-info.ts
        │   ├── contact-form-labels.ts
        │   ├── general-inquiry-labels.ts
        │   ├── header.ts
        │   └── footer.ts
        └── shared/
            ├── link.ts
            ├── menu-link.ts
            ├── feature.ts
            ├── cta-feature.ts
            ├── contact.ts
            └── hours.ts
```

And in the Next.js app:

```
frontend/src/app/
└── studio/
    └── [[...tool]]/
        └── page.tsx           # Studio route
```

### Pattern 1: Schema Definition with `defineType`

```typescript
// Source: https://www.sanity.io/docs/nextjs/embedding-sanity-studio-in-nextjs
// frontend/src/sanity/schemas/documents/home-page.ts
import { defineField, defineType } from "sanity";

export const homePage = defineType({
  name: "homePage",
  title: "Home Page",
  type: "document",
  fields: [
    defineField({ name: "title", type: "string" }),
    defineField({ name: "description", type: "text" }),
    defineField({
      name: "blocks",
      type: "array",
      of: [
        { type: "heroSection" },
        { type: "featuresSection" },
      ],
    }),
  ],
});
```

Named export (`export const homePage`) matches the project convention: kebab-case file, camelCase named export.

### Pattern 2: Studio Route (INFRA-01)

```typescript
// Source: https://www.sanity.io/docs/nextjs/embedding-sanity-studio-in-nextjs
// frontend/src/app/studio/[[...tool]]/page.tsx
import { NextStudio } from "next-sanity/studio";
import config from "../../../sanity/sanity.config";

export const dynamic = "force-static";
export { metadata, viewport } from "next-sanity/studio";

export default function StudioPage() {
  return <NextStudio config={config} />;
}
```

Key points:
- `export const dynamic = "force-static"` — static rendering for Studio; required
- `export { metadata, viewport } from "next-sanity/studio"` — sets mobile meta tags and disables crawling
- `[[...tool]]` is an optional catch-all that captures all Studio sub-routes (structure, vision)
- No `"use client"` needed on the page; `NextStudio` handles it internally
- `sanity.config.ts` must have `'use client'` at the top

### Pattern 3: `sanity.config.ts` with Singleton Pattern

```typescript
// Source: https://www.sanity.io/guides/singleton-document
// frontend/src/sanity/sanity.config.ts
'use client'

import { defineConfig } from "sanity";
import { structureTool } from "sanity/structure";
import { visionTool } from "@sanity/vision";
import { schemaTypes } from "./schemas";
import { structure } from "./structure";

const singletonTypes = new Set([
  "homePage", "contactPage", "galleryPage", "trainingPage",
  "trainingProgramsPage", "globalSettings", "mainMenu",
]);

const singletonActions = new Set(["publish", "discardChanges", "restore"]);

export default defineConfig({
  name: "default",
  title: "Lash Her by Nataliea",
  basePath: "/studio",
  projectId: process.env.NEXT_PUBLIC_SANITY_PROJECT_ID!,
  dataset: process.env.NEXT_PUBLIC_SANITY_DATASET!,
  apiVersion: process.env.NEXT_PUBLIC_SANITY_API_VERSION ?? "2026-03-24",
  plugins: [
    structureTool({ structure }),
    visionTool(),
  ],
  schema: {
    types: schemaTypes,
    // Prevent "New document" creation of singletons
    templates: (templates) =>
      templates.filter(({ schemaType }) => !singletonTypes.has(schemaType)),
  },
  document: {
    // Restrict actions for singletons (no delete, duplicate)
    actions: (input, context) =>
      singletonTypes.has(context.schemaType)
        ? input.filter(({ action }) => action && singletonActions.has(action))
        : input,
  },
});
```

### Pattern 4: Structure Builder — 3-Section Sidebar (D-01, D-02, D-03)

```typescript
// Source: https://www.sanity.io/docs/studio/create-a-link-to-a-single-edit-page-in-your-main-document-type-list
// frontend/src/sanity/structure/index.ts
import {
  StructureResolver,
  StructureBuilder,
} from "sanity/structure";
import {
  HomeIcon, EnvelopeIcon, PhotoIcon, AcademicCapIcon,
  BookOpenIcon, Cog6ToothIcon, Bars3Icon,
  // ... lucide or hero icons work here
} from "@sanity/icons";

export const structure: StructureResolver = (S: StructureBuilder) =>
  S.list()
    .title("Lash Her")
    .items([
      // ---- PAGES section ----
      S.listItem().title("Pages").child(
        S.list().title("Pages").items([
          S.listItem()
            .title("Homepage")
            .child(S.document().schemaType("homePage").documentId("homePage")),
          S.listItem()
            .title("Contact Page")
            .child(S.document().schemaType("contactPage").documentId("contactPage")),
          S.listItem()
            .title("Gallery")
            .child(S.document().schemaType("galleryPage").documentId("galleryPage")),
          S.listItem()
            .title("Training")
            .child(S.document().schemaType("trainingPage").documentId("trainingPage")),
          S.listItem()
            .title("Training Programs Overview")
            .child(S.document().schemaType("trainingProgramsPage").documentId("trainingProgramsPage")),
          S.listItem()
            .title("Global Settings")
            .child(S.document().schemaType("globalSettings").documentId("globalSettings")),
          S.listItem()
            .title("Navigation Menu")
            .child(S.document().schemaType("mainMenu").documentId("mainMenu")),
        ])
      ),
      S.divider(),
      // ---- CONTENT section ----
      S.listItem().title("Content").child(
        S.list().title("Content").items([
          S.documentTypeListItem("trainingProgram").title("Training Programs"),
        ])
      ),
      S.divider(),
      // ---- SUBMISSIONS section ----
      S.listItem().title("Submissions").child(
        S.list().title("Submissions").items([
          S.documentTypeListItem("generalInquiry").title("General Inquiries"),
          S.documentTypeListItem("contactForm").title("Training Contact Forms"),
        ])
      ),
    ]);
```

Note: The D-02 decision ("singletons open directly when clicked") applies at the individual singleton level. The 3-section grouping is a parent list item that leads to a child list of singletons. Within that child list, each singleton opens directly via `S.document().schemaType(...).documentId(...)`. If a truly flat sidebar is preferred (all 7 singletons visible at the top level with no nested pages group), adjust the structure — but the 3-section grouped approach above matches D-01 most naturally.

### Pattern 5: Read Client (INFRA-03)

```typescript
// Source: https://www.sanity.io/docs/nextjs/configure-sanity-client-nextjs
// frontend/src/sanity/lib/client.ts
import { createClient } from "next-sanity";
import { apiVersion, dataset, projectId } from "../env";

export const client = createClient({
  projectId,
  dataset,
  apiVersion,
  useCdn: true,
});
```

### Pattern 6: Write Client (INFRA-04)

```typescript
// Source: https://www.sanity.io/docs/apis-and-sdks/js-client-getting-started
// frontend/src/sanity/lib/write-client.ts
import "server-only";  // prevents import in client bundles
import { createClient } from "@sanity/client";
import { apiVersion, dataset, projectId } from "../env";

export const writeClient = createClient({
  projectId,
  dataset,
  apiVersion,
  useCdn: false,           // never use CDN for writes
  token: process.env.SANITY_WRITE_TOKEN,
});
```

The `import "server-only"` directive causes a build error if this module is accidentally imported in a client component — the correct enforcement mechanism in Next.js App Router.

### Pattern 7: Environment Variables Config Module

```typescript
// frontend/src/sanity/env.ts
export const apiVersion =
  process.env.NEXT_PUBLIC_SANITY_API_VERSION || "2026-03-24";

export const dataset = assertValue(
  process.env.NEXT_PUBLIC_SANITY_DATASET,
  "Missing env var: NEXT_PUBLIC_SANITY_DATASET"
);

export const projectId = assertValue(
  process.env.NEXT_PUBLIC_SANITY_PROJECT_ID,
  "Missing env var: NEXT_PUBLIC_SANITY_PROJECT_ID"
);

function assertValue<T>(v: T | undefined, errorMessage: string): T {
  if (v === undefined) throw new Error(errorMessage);
  return v;
}
```

### Pattern 8: Portable Text Field (D-13)

Fields that use `blocks` in Strapi map to Portable Text arrays in Sanity. The schema defines the Portable Text field now; the renderer is built in Phase 3.

```typescript
// Used in: imageWithText.perks, infoSection.info, ctaFeature.features
defineField({
  name: "content",   // or "perks", "info", "features" depending on context
  title: "Content",
  type: "array",
  of: [
    {
      type: "block",
      styles: [
        { title: "Normal", value: "normal" },
        { title: "H2", value: "h2" },
        { title: "H3", value: "h3" },
      ],
      lists: [
        { title: "Bullet", value: "bullet" },
        { title: "Numbered", value: "number" },
      ],
      marks: {
        decorators: [
          { title: "Strong", value: "strong" },
          { title: "Emphasis", value: "em" },
        ],
        annotations: [
          {
            name: "link",
            type: "object",
            title: "Link",
            fields: [
              { name: "href", type: "string", title: "URL" },
              { name: "blank", type: "boolean", title: "Open in new tab" },
            ],
          },
        ],
      },
    },
  ],
});
```

### Pattern 9: Image Field with Hotspot + Alt Text

```typescript
// Used in: heroSection.image, imageWithText.image, ctaSectionImage.image
defineField({
  name: "image",
  title: "Image",
  type: "image",
  options: { hotspot: true },
  fields: [
    defineField({
      name: "alt",
      title: "Alternative Text",
      type: "string",
      description: "Describe the image for accessibility",
    }),
  ],
})
```

### Pattern 10: Polymorphic Array (SCHEMA-05 — Navigation Menu)

The Strapi `main-menu` uses a dynamic zone with `menu.dropdown` and `menu.menu-link`. In Sanity, this maps to an array field with two allowed object types.

```typescript
// frontend/src/sanity/schemas/documents/main-menu.ts
export const mainMenu = defineType({
  name: "mainMenu",
  title: "Navigation Menu",
  type: "document",
  fields: [
    defineField({
      name: "items",
      title: "Menu Items",
      type: "array",
      of: [
        { type: "menuDirectLink" },    // maps to menu.menu-link (direct link)
        { type: "menuDropdown" },      // maps to menu.dropdown (multi-section)
      ],
    }),
  ],
});
```

The `menuDirectLink` object has `{ title, url }` fields; `menuDropdown` has `{ title, sections[] }` where each section has a title and array of `menuLink` sub-objects (with `name`, `url`, `description`, `icon`).

### Anti-Patterns to Avoid

- **Do not use `deskTool` from `sanity/desk`** — it was renamed to `structureTool` from `sanity/structure` in Sanity v3.24.1+. The current version (5.18.0) uses `structureTool`.
- **Do not add `'use client'` to schema files** — only `sanity.config.ts` needs it.
- **Do not prefix `SANITY_WRITE_TOKEN` with `NEXT_PUBLIC_`** — write tokens must never be exposed in client bundles.
- **Do not import write client in `page.tsx` or client components** — only in Server Actions, API routes, or Server Components. The `server-only` import enforces this at build time.
- **Do not skip `export { metadata, viewport } from "next-sanity/studio"` on the Studio page** — without it, mobile rendering and SEO behavior of the Studio page are broken.
- **Do not use `S.documentTypeListItem()` for singletons** — that shows a list of all documents of that type, not the single document. Use `S.document().schemaType().documentId()`.

---

## Complete Strapi-to-Sanity Schema Mapping

This is the authoritative mapping derived from reading all Strapi schemas. Use this as the ground truth for schema implementation.

### Document Types (10 total)

| Sanity `_type` | Strapi Source | Kind | Notes |
|----------------|---------------|------|-------|
| `homePage` | `home-page` | singleton | blocks: `heroSection`, `featuresSection` |
| `contactPage` | `contact` | singleton | blocks: `contactInfo`, `schedule`, `contactFormLabels`, `generalInquiryLabels` |
| `galleryPage` | `gallery` | singleton | blocks: `photoGallery`, `heroSection` |
| `trainingPage` | `training` | singleton | blocks: `ctaFeaturesSection`, `imageWithText` |
| `trainingProgramsPage` | `training-programs-page` | singleton | no blocks; references `trainingProgram[]` |
| `globalSettings` | `global` | singleton | has `header` and `footer` as nested objects (not blocks array) |
| `mainMenu` | `main-menu` | singleton | polymorphic `items[]` array |
| `trainingProgram` | `training-program` | collection | slug field, blocks: `heroSection`, `infoSection`, `contactFormLabels` |
| `contactForm` | `contact-form` | collection | submission; no `draftAndPublish` |
| `generalInquiry` | `general-inquiry` | collection | submission; no `draftAndPublish` |

**Note on `trainingProgramsPage`:** Strapi has a `oneToMany` relation to `training-program`. In Sanity, this is modeled as an array of references: `defineField({ name: "trainingPrograms", type: "array", of: [{ type: "reference", to: [{ type: "trainingProgram" }] }] })`.

### Layout Object Types (14 total — SCHEMA-03)

| Sanity `_type` | Strapi Source | Key Fields |
|----------------|---------------|------------|
| `heroSection` | `layout.hero-section` | `image`, `heading`, `subHeading`, `description`, `link[]` (link objects), `onHomepage` (bool) |
| `featuresSection` | `layout.features-section` | `title`, `heading`, `subHeading`, `description`, `features[]` (feature objects) |
| `ctaFeaturesSection` | `layout.cta-features-section` | `heading`, `subHeading`, `description`, `features[]` (ctaFeature objects) |
| `ctaSectionImage` | `layout.cta-section-image` | `heading`, `description`, `image`, `link[]` |
| `ctaSectionVideo` | `layout.cta-section-video` | `title`, `description`, `video` (file type), `link[]` |
| `imageWithText` | `layout.image-with-text` | `heading`, `subHeading`, `description`, `perks` (Portable Text), `image`, `orientation` (enum) |
| `infoSection` | `layout.info-section` | `heading`, `subHeading`, `info` (Portable Text) |
| `photoGallery` | `layout.photo-gallery` | `heading`, `subHeading`, `description`, `images[]` (image array with hotspot) |
| `schedule` | `layout.schedule` | `heading`, `subHeading`, `hours[]` (hours objects) |
| `contactInfo` | `layout.contact-info` | `heading`, `subHeading`, `contact[]` (contact objects) |
| `contactFormLabels` | `layout.contact-form` | `heading`, `subHeading`, label strings for each form field |
| `generalInquiryLabels` | `layout.general-inquiry-labels` | `heading`, `subHeading`, label strings for each form field |
| `header` | `layout.header` | `logoText` (link object), `ctaButton[]` (link array) |
| `footer` | `layout.footer` | `logoText` (link object), `text`, `socialLink[]` (link array) |

**Note on `imageWithText.orientation`:** Strapi enum values are `HORIZONTAL_IMAGE_LEFT`, `HORIZONTAL_IMAGE_RIGHT`, `VERTICAL`. Mirror exactly in Sanity using `list: [{ title: 'Image Left', value: 'HORIZONTAL_IMAGE_LEFT' }, ...]`.

**Note on `ctaSectionVideo.video`:** Strapi uses `type: media, allowedTypes: [videos]`. In Sanity, use `type: "file", options: { accept: "video/*" }`.

**Note on `photoGallery.image`:** Strapi allows both images and videos (`allowedTypes: ["images", "videos"]`). In Sanity, use an array that allows both `image` and `file` types, or restrict to images only if videos aren't actually used in practice.

### Reusable Sub-Object Types (6 total — SCHEMA-04)

| Sanity `_type` | Strapi Source | Fields |
|----------------|---------------|--------|
| `link` | `components.link` | `href` (string), `label` (string), `isExternal` (bool, default false) |
| `menuLink` | `components.menu-link` | `name`, `url`, `description`, `icon` (image) |
| `feature` | `components.feature` | `heading`, `subHeading` (text), `icon` (enum: EYE_ICON, SPARKLES_ICON, STAR_ICON) |
| `ctaFeature` | `components.cta-feature` | `heading`, `subHeading`, `location`, `tier`, `features` (Portable Text), `link` (link object), `icon` (enum), `mostPopular` (bool) |
| `contact` | `components.contact` | `phone`, `email`, `location` |
| `hours` | `components.hours` | `days`, `times` |

### Navigation-Specific Types (maps to Strapi `menu.*`)

| Sanity `_type` | Strapi Source | Fields |
|----------------|---------------|--------|
| `menuDirectLink` | `menu.menu-link` | `title` (string), `url` (string) |
| `menuDropdown` | `menu.dropdown` | `title` (string), `sections[]` (array of menuDropdownSection objects) |

**Note on `menu.dropdown.sections`:** In Strapi, `sections` is a relation to `api::section.section`. Since D-11 skips the `section` collection type entirely, this must be modeled differently in Sanity. The `sections` field becomes an inline array of objects containing `{ title, links[] }` where each link is a `menuLink` object. This is a deliberate schema improvement that avoids the unused `section` collection entirely.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Block content rich text | Custom text format | `type: "array", of: [{ type: "block" }]` (Portable Text) | Portable Text is the Sanity standard; custom formats lack renderer ecosystem |
| Image URL construction with crop/hotspot | Manual CDN URL assembly | `@sanity/image-url` (Phase 2) | Hotspot math and transform params are non-trivial; the library handles them correctly |
| Singleton enforcement | Custom validation | `schema.templates` filter + `document.actions` restriction in `defineConfig` | Sanity's built-in mechanism; no plugin needed |
| Studio route setup | Custom React app mount | `NextStudio` from `next-sanity/studio` | Handles metadata, mobile layout, and Next.js font/CSS compatibility |
| Client-side env var validation | Runtime `if (!process.env.X)` checks | `assertValue()` utility in `env.ts` + TypeScript `!` non-null assertion | Fail fast at startup rather than at query time |

---

## Common Pitfalls

### Pitfall 1: Using `deskTool` instead of `structureTool`
**What goes wrong:** `import { deskTool } from "sanity/desk"` — this is the old v3 import path, renamed in v3.24.1. The current `sanity` package (5.x) exports `structureTool` from `"sanity/structure"`.
**Why it happens:** Most tutorials online predate the rename; many Stack Overflow answers and older blog posts still show `deskTool`.
**How to avoid:** Always import `structureTool` from `"sanity/structure"`.
**Warning signs:** TypeScript error on import or deprecation warning at Studio startup.

### Pitfall 2: Forgetting `'use client'` on `sanity.config.ts`
**What goes wrong:** Studio fails to mount; React Server Component tries to render client-side Sanity Studio code.
**Why it happens:** Sanity Studio is a React SPA running inside Next.js — it cannot be a Server Component.
**How to avoid:** First line of `sanity.config.ts` must be `'use client'`.
**Warning signs:** Hydration errors or blank Studio page.

### Pitfall 3: Singleton created without initial document
**What goes wrong:** After configuring `schema.templates` filter, there is no way to create the singleton document through the UI. The Studio opens an empty editor with no document to save to.
**Why it happens:** The template filter prevents creation but the document doesn't exist yet.
**How to avoid:** Either (a) create the singleton documents via `npx sanity dataset import` before filtering, or (b) temporarily remove the filter during initial setup, create the documents, then add the filter back. Alternatively, create them programmatically via the write client.
**Warning signs:** Studio shows "0 documents" when navigating to a singleton type directly.

### Pitfall 4: `documentId` mismatch in structure vs schema
**What goes wrong:** `S.document().schemaType("homePage").documentId("homePage")` uses a hardcoded ID `"homePage"`. If a different ID is used anywhere (e.g., created manually with a generated ID), the structure builder opens the wrong document or creates a new one.
**Why it happens:** Sanity allows any string as `documentId`; the structure builder uses whatever ID is specified.
**How to avoid:** Use the same string for `documentId` in the structure builder as the `_type` name itself (e.g., `"homePage"`). Document that convention explicitly. All singleton IDs = their `_type` name.

### Pitfall 5: Write token visible in client bundle
**What goes wrong:** Prefixing `SANITY_WRITE_TOKEN` with `NEXT_PUBLIC_` exposes it in the browser bundle. Anyone can make authenticated writes to the Sanity dataset.
**Why it happens:** Developers follow the pattern from `NEXT_PUBLIC_SANITY_PROJECT_ID` without understanding which vars must be private.
**How to avoid:** `SANITY_WRITE_TOKEN` must never have `NEXT_PUBLIC_` prefix. The `import "server-only"` in `write-client.ts` causes a build error if the module is imported in a client component.

### Pitfall 6: Schema barrel file (`schemas/index.ts`) missing a new type
**What goes wrong:** Schema type is defined but not added to the `schemaTypes` array exported from `schemas/index.ts`. The type doesn't appear in Studio, but TypeScript doesn't catch the omission.
**Why it happens:** With 20+ schema files, it's easy to add a file without updating the barrel.
**How to avoid:** The barrel file must explicitly list all schemas. Consider grouping imports by category with comments. The planner should treat the barrel update as its own task step.
**Warning signs:** Type doesn't appear in Studio's block picker or document type list.

### Pitfall 7: `menu.dropdown.sections` relation skipped without a replacement
**What goes wrong:** Strapi's `menu.dropdown.sections` is a relation to the `section` collection (which is skipped, D-11). If the field is simply copied as a reference array pointing to `section` documents, it references a non-existent collection.
**Why it happens:** Mechanical translation of Strapi schemas without considering that the `section` collection is out of scope.
**How to avoid:** Replace with an inline array of section objects directly in the `menuDropdown` schema. See the schema mapping table above.

### Pitfall 8: `photo-gallery` image array allows videos — but Next.js Image doesn't
**What goes wrong:** Strapi allows images and videos in the gallery. If videos are stored as Sanity `image` type assets, they fail to render. If the frontend uses Next.js `<Image>`, video URLs crash.
**Why it happens:** Direct translation of `allowedTypes: ["images", "videos"]`.
**How to avoid:** Research whether the current gallery actually contains videos (likely not). Default to image-only in the schema. If videos are needed, use `type: "file"` with `options: { accept: "video/*" }` as a separate array member, and handle differently in the renderer.

---

## Environment Availability

| Dependency | Required By | Available | Version | Fallback |
|------------|-------------|-----------|---------|---------|
| Node.js | All JS tooling | ✓ | 22.21.0 | — |
| npm | Package installation | ✓ | 11.6.2 | — |
| `@sanity/cli` (global) | `npx sanity` commands | ✓ | 5.14.1 | Use `npx sanity@latest` |
| Sanity account + project | INFRA-02 project creation | ✗ (manual) | — | Must be created at sanity.io/manage before env vars can be populated |
| `NEXT_PUBLIC_SANITY_PROJECT_ID` | All clients + Studio | ✗ (not yet set) | — | Phase cannot complete without this — requires Sanity project creation |
| `NEXT_PUBLIC_SANITY_DATASET` | All clients + Studio | ✗ (not yet set) | — | — |
| `SANITY_WRITE_TOKEN` | Write client | ✗ (not yet set) | — | Read-only operations work without it |

**Missing dependencies with no fallback:**
- Sanity project must be created at `sanity.io/manage` before any GROQ queries or Studio access can be tested. This is a manual prerequisite step that cannot be automated without credentials.

**Missing dependencies with fallback:**
- `SANITY_WRITE_TOKEN` — Phase 1 can be completed without it (write client exists but isn't called in Phase 1). Required for Phase 4 (forms).

---

## INFRA-02: Two Projects vs Two Datasets

**Locked decision (from STATE.md):** "Two Sanity projects required (staging + production) — free tier allows only one dataset per project."

**Current pricing reality (verified 2026-03-24):** The Sanity free tier now allows 2 datasets per project (public only). This means staging + production could live as two datasets in a single project.

**Research verdict:** The locked decision (two separate projects) is more conservative and remains valid even if technically unnecessary. Two separate projects also provide cleaner isolation of API tokens, project IDs, and team members. The plan should implement two separate projects as decided.

**Implication for the plan:** INFRA-02 is a manual prerequisite. The developer must create two Sanity projects at `sanity.io/manage` before the implementation tasks can be validated. The plan should include this as a Wave 0 prerequisite check.

---

## Validation Architecture

### Test Framework
| Property | Value |
|----------|-------|
| Framework | Playwright 1.58.2 |
| Config file | `frontend/playwright.config.ts` |
| Quick run command | `cd frontend && npx playwright test tests/studio.spec.ts --project=chromium` |
| Full suite command | `cd frontend && npx playwright test` |

### Phase Requirements → Test Map

| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| INFRA-01 | `/studio` loads Sanity Studio | smoke (E2E) | `npx playwright test tests/studio.spec.ts -x --project=chromium` | ❌ Wave 0 |
| INFRA-01 | All 7 singleton documents visible in Studio sidebar | smoke (E2E) | same test file | ❌ Wave 0 |
| INFRA-01 | `trainingProgram` collection visible in Studio sidebar | smoke (E2E) | same test file | ❌ Wave 0 |
| SCHEMA-01/02/03/04/05 | Schema types registered — block picker shows all layout types | smoke (E2E) | same test file | ❌ Wave 0 |
| INFRA-03 | Read client resolves in a Server Component without error | unit/integration | `cd frontend && npx tsc --noEmit` (type check) | ❌ Wave 0 |
| INFRA-04 | Write client is importable only in server-only modules | build-time | `cd frontend && npm run build` (build error if misused) | N/A — enforced by `server-only` |

**Note on SCHEMA validation:** Schema correctness (correct fields, types, enumerations) cannot be fully automated without a live Sanity dataset. The E2E tests validate that the Studio mounts and shows the expected document types. Full field validation is manual in Wave 0.

**Note on GROQ query validation (success criterion 4):** Running a GROQ query against the staging dataset requires a live dataset with credentials. This is a manual success criterion check — the automated test can only verify Studio loads.

### Sampling Rate
- **Per task commit:** `cd frontend && npx tsc --noEmit` (fast TypeScript check, ~5s)
- **Per wave merge:** `cd frontend && npx playwright test tests/studio.spec.ts --project=chromium`
- **Phase gate:** Full Studio E2E test passing before `/gsd:verify-work`

### Wave 0 Gaps
- [ ] `frontend/tests/studio.spec.ts` — Studio smoke tests (INFRA-01 + sidebar visibility)
- [ ] `frontend/src/app/studio/[[...tool]]/page.tsx` — Studio route (created as part of implementation)
- [ ] `frontend/src/sanity/` directory — entire Sanity module (created as part of implementation)
- [ ] Sanity projects at `sanity.io/manage` — manual prerequisite (two projects, get project IDs + write tokens)
- [ ] `frontend/.env.local` additions: `NEXT_PUBLIC_SANITY_PROJECT_ID`, `NEXT_PUBLIC_SANITY_DATASET`, `NEXT_PUBLIC_SANITY_API_VERSION`, `SANITY_WRITE_TOKEN`

---

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| `deskTool` from `sanity/desk` | `structureTool` from `sanity/structure` | Sanity v3.24.1 (2023) | Import path change; old path still works but deprecated |
| `__experimental_actions` in schema | `schema.templates` + `document.actions` in `defineConfig` | Sanity v3 (2022) | Cleaner singleton enforcement; no schema-level hack |
| `@sanity/vision` as separate plugin install | Ships with `sanity` package | Sanity v3+ | `visionTool()` available without extra install |
| `createClient` from `@sanity/client` | `createClient` from `next-sanity` for read client | next-sanity v6+ | `next-sanity` version adds Next.js cache tag integration |

**Deprecated/outdated:**
- `deskTool` from `sanity/desk`: works but deprecated — use `structureTool` from `sanity/structure`
- `S.defaults()`: was the old way to get auto-generated list items — now use `S.documentTypeListItems()` filtered

---

## Open Questions

1. **`menu.dropdown.sections` — actual content in live Strapi database**
   - What we know: Strapi schema defines `sections` as a relation to `api::section.section`; the `section` collection type is unused per D-11
   - What's unclear: Does the live Strapi data have any dropdown items with populated sections? If yes, what does the section data look like? This affects whether the inline sections model (proposed above) matches the actual data.
   - Recommendation: Before migration (Phase 6), query Strapi for `GET /api/main-menu?populate=deep` to see actual menu data shape.

2. **`photoGallery` — are videos actually present?**
   - What we know: Strapi schema allows both images and videos (`allowedTypes: ["images", "videos"]`)
   - What's unclear: Whether any gallery items in the live dataset contain videos
   - Recommendation: Query Strapi `GET /api/gallery?populate=deep` before finalizing the `photoGallery` schema. If videos are absent, use image-only schema. Flag in Phase 6 migration script.

3. **Singleton document creation order**
   - What we know: With `schema.templates` filter active, the Studio UI cannot create new singleton documents
   - What's unclear: The best ordering — create documents via Studio first (before filter), or via CLI/API, or as part of migration script in Phase 6?
   - Recommendation: Plan should include a task to create empty singleton documents (via `sanity documents create`) after projects are set up, before filtering is enabled. Alternatively, include singleton document initialization in the Phase 6 migration script.

4. **`NEXT_PUBLIC_SANITY_API_VERSION` — static date vs env var**
   - What we know: Sanity recommends a static date string (`"2026-03-24"`) for production stability
   - What's unclear: Whether this should be an env var at all (it's not secret and changing it is a code change anyway)
   - Recommendation: Hard-code `"2026-03-24"` as the default in `env.ts` with the env var as an optional override. Documented in the env module comments.

---

## Sources

### Primary (HIGH confidence)
- Official Sanity docs — https://www.sanity.io/docs/nextjs/embedding-sanity-studio-in-nextjs — Studio route setup, `force-static`, `metadata` export
- Official Sanity docs — https://www.sanity.io/docs/nextjs/configure-sanity-client-nextjs — Read client configuration, env vars
- Official Sanity docs — https://www.sanity.io/docs/apis-and-sdks/js-client-getting-started — Write client with `token`, `useCdn: false`
- Official Sanity guide — https://www.sanity.io/guides/singleton-document — Singleton pattern with `schema.templates` + `document.actions`
- Official Sanity docs — https://www.sanity.io/docs/studio/create-a-link-to-a-single-edit-page-in-your-main-document-type-list — `S.document().schemaType().documentId()` pattern
- Official Sanity docs — https://www.sanity.io/docs/studio/block-type — Portable Text schema definition
- Official Sanity docs — https://www.sanity.io/docs/studio/image-type — Image schema with hotspot
- Strapi source schemas — `backend/src/api/*/content-types/*/schema.json` — All 10 document types (read directly from codebase)
- Strapi component schemas — `backend/src/components/layout/*.json`, `components/*.json`, `menu/*.json` — All 20 component types

### Secondary (MEDIUM confidence)
- Sanity pricing page — https://www.sanity.io/pricing — Free tier: 2 datasets (public only) per project, 10k documents, 1M CDN requests/month
- npm registry `npm view` commands — Confirmed package versions: `sanity@5.18.0`, `next-sanity@12.1.6`, `@sanity/client@7.20.0`
- Sanity community — https://www.sanity.io/answers/any-hints-on-creating-a-group-of-singletons-p1610591053010300 — Grouped singleton structure builder pattern

### Tertiary (LOW confidence)
- None — all critical claims verified with official sources

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — versions confirmed via `npm view` on 2026-03-24
- Architecture patterns: HIGH — all patterns from official Sanity docs
- Schema mapping: HIGH — derived directly from reading Strapi source schemas
- Pitfalls: HIGH — `deskTool`/`structureTool` rename and singleton creation order verified from official docs and release notes
- INFRA-02 dataset count: MEDIUM — pricing page confirmed 2 datasets on free tier; STATE.md decision to use 2 projects is locked but based on outdated info

**Research date:** 2026-03-24
**Valid until:** 2026-06-24 (stable ecosystem; Sanity v5 is current major; next-sanity v12 is current major)
