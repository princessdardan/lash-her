# Phase 6: Content Migration and Cleanup - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-03-29
**Phase:** 06-content-migration-and-cleanup
**Areas discussed:** Form submission history, Cutover & verification, Strapi connectivity, Cleanup boundaries

---

## Form Submission History

| Option | Description | Selected |
|--------|-------------|----------|
| Structural content only (Recommended) | Migrate pages, settings, training programs, menu, gallery. Skip form submissions — they're historical records that don't affect the live site. Simpler migration, fewer documents. | |
| Migrate everything | Include all historical generalInquiry and contactForm submissions. They'd appear in Studio under Submissions. Useful for a unified record of all past inquiries in Sanity. | ✓ |
| Migrate recent only | Include form submissions from the last N months. A middle ground — recent history preserved, old noise excluded. | |

**User's choice:** Migrate everything
**Notes:** User wants all historical form submissions preserved in Sanity for a complete record.

---

## Cutover & Verification

| Option | Description | Selected |
|--------|-------------|----------|
| Staging first, then production (Recommended) | Run migration against staging dataset → verify document counts + visual spot-check → run again against production dataset → deploy. Two-pass approach catches issues before they hit prod. | |
| Production direct | Run migration directly against the production Sanity dataset. Simpler, but no dry-run safety net. Content volume is small enough that manual verification is feasible. | ✓ |
| Staging only (manual prod) | Migrate to staging, verify thoroughly, then manually re-enter the small amount of content in production. Avoids running scripts against prod entirely. | |

**User's choice:** Production direct
**Notes:** Content volume is small — direct production migration is acceptable risk.

---

## Strapi Connectivity

| Option | Description | Selected |
|--------|-------------|----------|
| Live Strapi API (Recommended) | Script hits the running Strapi REST API (e.g., http://localhost:1337 or Strapi Cloud URL). Strapi needs to be running during migration. Cleanest approach. | |
| Strapi Cloud hosted | Strapi is deployed on Strapi Cloud with a public URL. Script uses that URL + API token. No need to run anything locally. | ✓ |
| Local database export | Export the SQLite/Postgres database and read directly. Avoids needing Strapi running, but parsing raw DB is more complex than using the REST API. | |

**User's choice:** Strapi Cloud hosted
**Notes:** Strapi instance is on Strapi Cloud with a public API endpoint. Migration script uses the cloud URL + API token.

---

## Cleanup Boundaries

| Option | Description | Selected |
|--------|-------------|----------|
| Full removal (Recommended) | Delete backend/ entirely, remove all strapi-*.tsx files, old image scripts, Vercel Blob refs, error-handler.ts Strapi code. Everything is in git history if ever needed. | ✓ |
| Keep backend as archive | Delete all Strapi code from frontend/ but keep backend/ directory intact as a reference archive. | |
| Minimal cleanup | Only remove what breaks or causes confusion. Keep backend/ and old scripts but remove dead imports and type references from active code. | |

**User's choice:** Full removal (Recommended)
**Notes:** Scorched earth — everything Strapi-related removed. Git history serves as the archive.

---

## Claude's Discretion

- Migration script architecture (single vs. modular)
- Rich text conversion (Strapi Blocks → Portable Text)
- Image migration strategy (batch vs. inline)
- `_key` generation approach
- Error handling and retry logic
- Whether `vercel-install.mjs` survives cleanup (it's Motion-related, not Strapi)
- Whether `error-handler.ts` gets deleted entirely or just stripped

## Deferred Ideas

None — discussion stayed within phase scope.
