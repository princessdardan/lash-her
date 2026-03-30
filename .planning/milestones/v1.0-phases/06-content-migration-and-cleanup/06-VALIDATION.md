---
phase: 6
slug: content-migration-and-cleanup
status: draft
nyquist_compliant: true
wave_0_complete: true
created: 2026-03-29
---

# Phase 6 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Playwright E2E + Node.js scripts |
| **Config file** | `frontend/playwright.config.ts` |
| **Quick run command** | `cd frontend && npx playwright test tests/homepage.spec.ts --project=chromium` |
| **Full suite command** | `cd frontend && npm test` |
| **Estimated runtime** | ~30 seconds |

---

## Sampling Rate

- **After every task commit:** Run `cd frontend && npx playwright test tests/homepage.spec.ts --project=chromium`
- **After every plan wave:** Run `cd frontend && npm test`
- **Before `/gsd:verify-work`:** Full suite must be green
- **Max feedback latency:** 30 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 06-01-01 | 01 | 1 | MIG-01, MIG-02, MIG-03, MIG-04, MIG-05 | script | `cd frontend && node -e "require('nanoid')" && test -f scripts/migrate-strapi-to-sanity.ts` | ✅ | pending |
| 06-01-02 | 01 | 1 | MIG-06 | checkpoint | `echo "Manual: user runs npm run migrate and reviews summary report"` | N/A | pending |
| 06-02-01 | 02 | 2 | CLEAN-01, CLEAN-03 | filesystem | `! test -f frontend/src/data/strapi-loaders.ts && ! test -d backend/` | ✅ | pending |
| 06-02-02 | 02 | 2 | CLEAN-02, CLEAN-04 | grep+build | `cd frontend && ! grep -q "getStrapiURL\|fetchStrapi" src/lib/utils.ts && npm run build` | ✅ | pending |
| 06-03-01 | 03 | 3 | MIG-03 | e2e | `cd frontend && npx playwright test tests/training-programs.spec.ts --project=chromium` | ✅ | pending |
| 06-03-02 | 03 | 3 | CLEAN-05 | grep+e2e | `cd frontend && npm test && ! grep -rq "strapi\|STRAPI" src/ --include="*.ts" --include="*.tsx"` | ✅ | pending |
| 06-03-03 | 03 | 3 | MIG-06 | checkpoint | `echo "Manual: user visually verifies all pages render correctly"` | N/A | pending |

*Status: pending | green | red | flaky*

---

## Wave 0 Requirements

No Wave 0 gaps. Plan 01 creates `frontend/scripts/migrate-strapi-to-sanity.ts` (TypeScript, run via `tsx`). Plan 03 creates `frontend/tests/training-programs.spec.ts` for MIG-03 rich text coverage.

*Existing Playwright infrastructure covers all E2E verification needs.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Visual parity with current site | MIG-05, MIG-06 | Requires human visual comparison | Compare each page against current production, check layout, images, text |
| Migration summary report review | MIG-06 | One-time script output review | Run `npm run migrate`, verify OK for all content types in terminal output |
| Vercel env var cleanup | CLEAN-05 | Requires Vercel dashboard access | Remove STRAPI env vars, verify Sanity vars present |

---

## Validation Sign-Off

- [x] All tasks have `<automated>` verify or checkpoint justification
- [x] Sampling continuity: no 3 consecutive tasks without automated verify
- [x] No Wave 0 gaps — all referenced files exist or are created by plan tasks
- [x] No watch-mode flags
- [x] Feedback latency < 30s
- [x] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
