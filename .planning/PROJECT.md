# Lash Her — Strapi to Sanity Migration

## What This Is

A full CMS migration of the Lash Her by Nataliea website from Strapi 5 to Sanity. The site is a content-driven marketing platform for a lash artistry and training business, built with Next.js 16 and currently powered by a Strapi 5 backend. This milestone replaces Strapi entirely with Sanity — content modeling, API layer, image pipeline, form submissions, email notifications, and content studio — while preserving every frontend feature and visual behavior.

## Core Value

Every page, form, image, and interaction that works today must work identically after migration — zero regression in the production user experience.

## Requirements

### Validated

- ✓ CMS-driven homepage with dynamic zone blocks — existing
- ✓ Contact page with general inquiry form and email notifications — existing
- ✓ Photo gallery page — existing
- ✓ Training information page — existing
- ✓ Dynamic training program pages with slug-based routing — existing
- ✓ Training contact form with email notifications — existing
- ✓ CMS-driven navigation menu (direct links + multi-section dropdowns) — existing
- ✓ Global site settings (metadata, footer content) — existing
- ✓ Block renderer pattern mapping CMS components to React components — existing
- ✓ Scroll-based entrance animations on layout blocks — existing
- ✓ Responsive header with scroll-hide behavior — existing
- ✓ ISR caching strategy with stale-while-revalidate — existing
- ✓ Image optimization via Next.js Image with remote patterns — existing
- ✓ SEO metadata from CMS — existing
- ✓ Vercel Analytics and Speed Insights integration — existing

### Active

- [x] Sanity schema design mirroring all Strapi content types and components — Validated in Phase 1
- [x] Sanity Studio embedded in Next.js at /studio route — Validated in Phase 1
- [ ] GROQ-based data loaders replacing Strapi REST API calls
- [ ] Sanity image pipeline replacing Strapi uploads and Vercel Blob
- [ ] Automated content migration scripts (Strapi → Sanity)
- [ ] Form submissions stored as Sanity documents
- [ ] Email notifications via Sanity webhook or serverless function (Resend)
- [ ] Block renderer adapted for Sanity portable text and object arrays
- [ ] Rich text rendering via Sanity's Portable Text
- [ ] Remove Strapi backend entirely after migration complete

### Out of Scope

- New features or pages — migration only, feature parity
- Strapi-to-Sanity real-time sync or dual-CMS operation — big bang cutover
- Mobile app or native clients — web only
- Redesign or visual changes — identical frontend appearance
- Paid Sanity features — staying on free tier

## Context

**Existing system:** Strapi 5 headless CMS with 7 single types (home-page, contact, gallery, global, main-menu, training, training-programs-page), 4 collection types (contact-form, general-inquiry, section, training-program), 14 layout components, 6 reusable sub-components, and 2 menu components. The frontend uses a unified API client (`data-api.ts`) with typed responses, centralized data loaders (`loaders.ts`), and a block renderer mapping Strapi dynamic zone `__component` values to React components.

**Migration drivers:**
1. **Sanity's editing UX** — real-time collaboration, structured content studio, better experience for content editors
2. **Scaling/hosting** — eliminate Strapi Cloud costs and self-hosting burden; Sanity's hosted content lake
3. **Developer experience** — GROQ queries, schema-as-code, better TypeScript support, content lake flexibility

**Cutover strategy:** Big bang — build everything against Sanity, migrate all content via automated scripts, deploy in one swap. The existing Strapi production site continues to function until the Sanity-backed version is fully ready.

**Content volume:** Small — a handful of pages, ~4 training programs, gallery images, form submissions. Manageable for automated migration scripts.

**Sanity plan:** Free tier (generous limits for this content volume).

## Constraints

- **Tech stack**: Next.js 16 (App Router), React 18, TypeScript, Tailwind CSS v4 — unchanged
- **Sanity Studio**: Embedded in Next.js via next-sanity (at /studio route)
- **Images**: All images migrate to Sanity CDN with hotspot/crop support
- **Forms**: Submissions stored as Sanity documents; emails triggered via webhook or serverless function using Resend
- **Compatibility**: Node >=20, npm workspaces retained for frontend (backend workspace removed post-migration)
- **Zero regression**: All existing pages, forms, animations, and interactions must work identically

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Big bang cutover (not incremental) | Content volume is small, project isn't high-traffic — simpler to build and swap | — Pending |
| Sanity Studio embedded in Next.js | Single deployment, shared auth context, simpler ops | ✓ Phase 1 |
| All images to Sanity CDN | Consolidate image sources, leverage Sanity's image pipeline (transforms, hotspot, CDN) | — Pending |
| Forms as Sanity documents + webhook emails | Keep form data in CMS, trigger emails externally — replaces Strapi lifecycle hooks | — Pending |
| Free tier Sanity | Content volume is small, free tier is sufficient | — Pending |
| Automated content migration scripts | Ensures accuracy and repeatability vs manual re-entry | — Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd:transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd:complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-03-25 after Phase 1 completion*
