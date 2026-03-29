# Requirements: Lash Her — Strapi to Sanity Migration

**Defined:** 2026-03-24
**Core Value:** Every page, form, image, and interaction that works today must work identically after migration — zero regression in the production user experience.

## v1 Requirements

Requirements for the complete Strapi 5 → Sanity CMS migration. Each maps to roadmap phases.

### Schema & Content Modeling

- [x] **SCHEMA-01**: Sanity schema defined for all 7 singleton documents (homePage, contactPage, galleryPage, trainingPage, trainingProgramsPage, globalSettings, mainMenu) using singleton pattern
- [x] **SCHEMA-02**: Sanity schema defined for collection types (contactForm, generalInquiry, trainingProgram)
- [x] **SCHEMA-03**: 14 layout object types mirroring Strapi layout components (heroSection, featuresSection, ctaFeaturesSection, imageWithText, infoSection, photoGallery, schedule, contactInfo, contactFormLabels, generalInquiryLabels, header, footer, ctaSectionImage, ctaSectionVideo)
- [x] **SCHEMA-04**: 6 reusable sub-object types (link, menuLink, feature, ctaFeature, contact, hours) defined once and referenced across document types
- [x] **SCHEMA-05**: Navigation menu schema supports polymorphic items — direct links and multi-section dropdowns with `_type` discriminator

### Data Layer

- [x] **DATA-01**: GROQ-based data loaders replace all 8 functions in `loaders.ts` (getHomePageData, getContactPageData, getGalleryPageData, getTrainingsPageData, getMainMenuData, getGlobalData, getMetaData, getTrainingProgramBySlug)
- [x] **DATA-02**: Block renderer `COMPONENT_REGISTRY` keys updated from Strapi `__component` format (`layout.hero-section`) to Sanity `_type` format (`heroSection`)
- [x] **DATA-03**: TypeScript types rewritten to match Sanity document shapes (replace `TStrapiResponse<T>` envelope pattern with direct GROQ result types)
- [x] **DATA-04**: `generateStaticParams` for training programs rewritten from hardcoded slugs to GROQ query
- [x] **DATA-05**: All 5 content pages (homepage, contact, gallery, training, training-programs/[slug]) render correctly from Sanity data

### Studio & Infrastructure

- [x] **INFRA-01**: Sanity Studio embedded in Next.js at `/studio` route via `next-sanity` `NextStudio` component with catch-all `[[...tool]]` routing
- [ ] **INFRA-02**: Two Sanity projects created — staging and production — to work within free tier dataset limits
- [ ] **INFRA-03**: Sanity read client configured (`useCdn: true`) for production content fetching
- [ ] **INFRA-04**: Sanity write client configured (server-only, `SANITY_WRITE_TOKEN`) for form submissions and migration scripts
- [x] **INFRA-05**: On-demand ISR via Sanity webhook calling `/api/revalidate` endpoint with `revalidateTag()` — content changes go live immediately on publish
- [x] **INFRA-06**: Webhook signature verification using shared secret (`SANITY_WEBHOOK_SECRET`)

### Images

- [x] **IMG-01**: `sanity-image.tsx` component replaces `strapi-image.tsx` using `@sanity/image-url` URL builder with Next.js `<Image>`
- [x] **IMG-02**: All images uploaded to Sanity CDN with hotspot and crop support enabled
- [x] **IMG-03**: `cdn.sanity.io` added to `next.config.ts` remote patterns; Strapi and Vercel Blob patterns removed after cutover
- [x] **IMG-04**: Sanity image transforms via URL params available (width, height, format, crop) — replaces optimize scripts

### Rich Text

- [ ] **RT-01**: Portable Text renderer built using `@portabletext/react` with brand-styled component map matching existing visual output (headings, paragraphs, lists, links, bold, italic)
- [ ] **RT-02**: Existing `ui/block-renderer.tsx` (rich text renderer) renamed to `strapi-rich-text-renderer.tsx` to eliminate naming collision before migration

### Forms & Email

- [ ] **FORM-01**: General inquiry form submits via Next.js Server Action → creates Sanity document + sends Resend admin notification and user confirmation emails in parallel
- [ ] **FORM-02**: Training contact form submits via Next.js Server Action → creates Sanity document + sends Resend admin notification and user confirmation emails in parallel
- [ ] **FORM-03**: Resend email service moved from `backend/src/services/email.ts` to `frontend/src/lib/email.ts` with equivalent HTML templates
- [ ] **FORM-04**: Write token scoped to create-only permissions on form submission document types

### Content Migration

- [ ] **MIG-01**: Automated migration script fetches all content from Strapi REST API, transforms to Sanity document shapes, and writes to Sanity content lake
- [ ] **MIG-02**: Migration script uploads all images from Strapi to Sanity CDN and updates document references to Sanity asset IDs
- [ ] **MIG-03**: Migration script converts Strapi Blocks rich text JSON to Sanity Portable Text format
- [ ] **MIG-04**: Migration script generates `_key` fields for all array items (nanoid) since Strapi uses integer IDs
- [ ] **MIG-05**: Migration script explicitly publishes all documents after creation (not left as drafts)
- [ ] **MIG-06**: Migration verified against production dataset — document counts, image rendering, and page content match source

### Cleanup

- [ ] **CLEAN-01**: Strapi backend workspace removed from `package.json` workspaces and `backend/` directory deleted
- [ ] **CLEAN-02**: Strapi-specific packages removed from frontend (`qs`, `@types/qs`)
- [ ] **CLEAN-03**: Strapi-specific code removed (old `data-api.ts`, `fetchStrapi()` in `utils.ts`, Strapi type definitions)
- [ ] **CLEAN-04**: Vercel Blob references removed (env var, remote pattern, migration scripts)
- [ ] **CLEAN-05**: Environment variables updated — Strapi vars removed, Sanity vars added to Vercel

## v2 Requirements

Deferred to post-migration. Tracked but not in current roadmap.

### Visual Editing

- **VE-01**: Visual Editing via `@sanity/presentation` plugin with stega-encoded overlay editing
- **VE-02**: Draft preview mode with `perspective: 'previewDrafts'`

### Developer Experience

- **DX-01**: TypeScript type generation via `@sanity/codegen` (`sanity typegen generate`)
- **DX-02**: Custom Sanity Studio plugins for enhanced editing UX

## Out of Scope

| Feature | Reason |
|---------|--------|
| New pages or features | Migration only — feature parity, not feature addition |
| Redesign or visual changes | Frontend appearance stays identical |
| Dual CMS operation | Big bang cutover, not incremental sync |
| Real-time GROQ subscriptions | ISR is sufficient for a marketing site |
| `section` collection type migration | Unused in current codebase (no loader, no frontend component) |
| Sanity paid features (Content Releases, etc.) | Free tier is sufficient for content volume |
| Custom `data-api.ts` response envelope | Sanity returns plain objects; no need to replicate `TStrapiResponse<T>` |
| Mobile app or native clients | Web only |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| SCHEMA-01 | Phase 1 | Complete |
| SCHEMA-02 | Phase 1 | Complete |
| SCHEMA-03 | Phase 1 | Complete |
| SCHEMA-04 | Phase 1 | Complete |
| SCHEMA-05 | Phase 1 | Complete |
| INFRA-01 | Phase 1 | Complete |
| INFRA-02 | Phase 1 | Pending |
| INFRA-03 | Phase 1 | Pending |
| INFRA-04 | Phase 1 | Pending |
| DATA-01 | Phase 2 | Complete |
| DATA-02 | Phase 2 | Complete |
| DATA-03 | Phase 2 | Complete |
| DATA-04 | Phase 2 | Complete |
| DATA-05 | Phase 2 | Complete |
| IMG-01 | Phase 2 | Complete |
| IMG-02 | Phase 2 | Complete |
| IMG-03 | Phase 2 | Complete |
| IMG-04 | Phase 2 | Complete |
| RT-01 | Phase 3 | Pending |
| RT-02 | Phase 3 | Pending |
| FORM-01 | Phase 4 | Pending |
| FORM-02 | Phase 4 | Pending |
| FORM-03 | Phase 4 | Pending |
| FORM-04 | Phase 4 | Pending |
| INFRA-05 | Phase 5 | Complete |
| INFRA-06 | Phase 5 | Complete |
| MIG-01 | Phase 6 | Pending |
| MIG-02 | Phase 6 | Pending |
| MIG-03 | Phase 6 | Pending |
| MIG-04 | Phase 6 | Pending |
| MIG-05 | Phase 6 | Pending |
| MIG-06 | Phase 6 | Pending |
| CLEAN-01 | Phase 6 | Pending |
| CLEAN-02 | Phase 6 | Pending |
| CLEAN-03 | Phase 6 | Pending |
| CLEAN-04 | Phase 6 | Pending |
| CLEAN-05 | Phase 6 | Pending |

**Coverage:**
- v1 requirements: 37 total
- Mapped to phases: 37
- Unmapped: 0

---
*Requirements defined: 2026-03-24*
*Last updated: 2026-03-24 after roadmap creation — all 37 requirements mapped*
