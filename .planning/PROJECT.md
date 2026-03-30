# Lash Her by Nataliea

## What This Is

A content-driven marketing website for a lash artistry and training business, built with Next.js 16 and powered by Sanity as the headless CMS. Features CMS-driven page composition via layout blocks, two contact forms with email notifications, a photo gallery, training program pages with rich text, and an embedded content studio at /studio.

## Core Value

Every page, form, image, and interaction that works today must work identically after migration — zero regression in the production user experience.

## Requirements

### Validated

- ✓ Sanity schema design mirroring all Strapi content types and components — v1.0
- ✓ Sanity Studio embedded in Next.js at /studio route — v1.0
- ✓ GROQ-based data loaders replacing Strapi REST API calls — v1.0
- ✓ Sanity image pipeline replacing Strapi uploads and Vercel Blob — v1.0
- ✓ On-demand cache revalidation via Sanity webhook + revalidateTag — v1.0
- ✓ Automated content migration scripts (Strapi → Sanity) — v1.0
- ✓ Form submissions stored as Sanity documents — v1.0
- ✓ Email notifications via Resend Server Actions — v1.0
- ✓ Block renderer adapted for Sanity portable text and object arrays — v1.0
- ✓ Rich text rendering via Sanity's Portable Text — v1.0
- ✓ Strapi backend removed entirely after migration — v1.0
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
- ✓ ISR caching strategy with on-demand revalidation — v1.0
- ✓ Image optimization via Next.js Image with Sanity CDN — v1.0
- ✓ SEO metadata from CMS — existing
- ✓ Vercel Analytics and Speed Insights integration — existing

### Active

(None — next milestone not yet planned)

### Out of Scope

- Visual Editing via @sanity/presentation plugin — deferred to v2 (VE-01, VE-02)
- TypeScript type generation via @sanity/codegen — deferred to v2 (DX-01)
- Custom Sanity Studio plugins — deferred to v2 (DX-02)
- Mobile app or native clients — web only
- Real-time GROQ subscriptions — ISR with on-demand revalidation is sufficient for a marketing site
- Paid Sanity features — free tier is sufficient for content volume

## Context

Shipped v1.0 with 6,932 LOC TypeScript across 26 files.
Tech stack: Next.js 16 (App Router), Sanity v3, Tailwind CSS v4, Radix UI, Motion, Resend, Playwright.
Content: 7 singleton pages, 3 collection types (training programs, form submissions), 33 Sanity schema types.
Deployed on Vercel with on-demand ISR via Sanity webhook.
Migration from Strapi 5 completed in 6 days with zero frontend regression.

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Big bang cutover (not incremental) | Content volume is small, project isn't high-traffic — simpler to build and swap | ✓ Good — completed in 6 days |
| Sanity Studio embedded in Next.js | Single deployment, shared auth context, simpler ops | ✓ Good — works at /studio |
| All images to Sanity CDN | Consolidate image sources, leverage Sanity's image pipeline (transforms, hotspot, CDN) | ✓ Good — all images serving from cdn.sanity.io |
| Forms as Sanity documents + Server Action emails | Keep form data in CMS, send emails via Resend in Server Actions — replaces Strapi lifecycle hooks | ✓ Good — reliable with Promise.allSettled |
| Free tier Sanity | Content volume is small, free tier is sufficient | ✓ Good — well within limits |
| Automated content migration scripts | Ensures accuracy and repeatability vs manual re-entry | ✓ Good — 1,259-line script handled all types |
| camelCase _type naming convention | Established in Phase 1 before any consumers written — consistent Sanity convention | ✓ Good — zero naming conflicts |
| Fetch-level cache tags (not unstable_cache) | Simpler caching model, tags applied directly to GROQ fetch calls | ✓ Good — eliminated TTL wrapper complexity |
| HMAC webhook signature verification | Prevents unauthorized cache purges, crashes at startup if secret missing | ✓ Good — security without silent degradation |
| Server Actions with module-level 'use server' | Required pattern for standalone action files in Next.js App Router | ✓ Good |

## Constraints

- **Tech stack**: Next.js 16 (App Router), React 18, TypeScript, Tailwind CSS v4 — unchanged
- **Sanity Studio**: Embedded in Next.js via next-sanity (at /studio route)
- **Images**: All images served from Sanity CDN with hotspot/crop support
- **Forms**: Submissions stored as Sanity documents; emails via Resend in Server Actions
- **Caching**: On-demand ISR via Sanity webhook + revalidateTag; fetch-level cache tags
- **Compatibility**: Node >=20, npm
- **Deployment**: Vercel

---
*Last updated: 2026-03-30 after v1.0 milestone*
