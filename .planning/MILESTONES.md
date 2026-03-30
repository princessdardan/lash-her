# Milestones

## v1.0 Strapi to Sanity Migration (Shipped: 2026-03-30)

**Phases completed:** 6 phases, 17 plans, 33 tasks
**Timeline:** 6 days (2026-03-24 → 2026-03-30)
**Commits:** 49 | **Files:** 26 changed | **LOC:** 6,932 TypeScript

**Key accomplishments:**

1. Defined 33 Sanity schemas (23 objects + 10 documents) and embedded Studio at /studio with custom 3-section sidebar
2. Built 10 GROQ-based data loaders, migrated all 11 layout components and 5 content pages to Sanity-shaped props
3. Created brand-styled Portable Text renderer with @portabletext/react and @tailwindcss/typography
4. Wired form submissions to Sanity documents + Resend email notifications via Server Actions (4 HTML templates)
5. Implemented on-demand ISR with Sanity webhook, HMAC-SHA256 signature verification, and fetch-level cache tags
6. Automated Strapi→Sanity content migration (1,259-line script with image upload + Portable Text conversion), then removed all Strapi code and backend directory

**Delivered:** Complete CMS migration from Strapi 5 to Sanity with zero frontend regression — every page, form, image, and interaction works identically.

**Archive:** [v1.0-ROADMAP.md](milestones/v1.0-ROADMAP.md) | [v1.0-REQUIREMENTS.md](milestones/v1.0-REQUIREMENTS.md)

---
