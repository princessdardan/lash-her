---
phase: 3
slug: rich-text-and-portable-text-renderer
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-26
---

# Phase 3 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Playwright 1.58.2 |
| **Config file** | `frontend/playwright.config.ts` |
| **Quick run command** | `cd frontend && npm run build` |
| **Full suite command** | `cd frontend && npx playwright test` |
| **Estimated runtime** | ~30 seconds (build), ~60 seconds (Playwright) |

---

## Sampling Rate

- **After every task commit:** Run `cd frontend && npm run build`
- **After every plan wave:** Run `cd frontend && npx playwright test tests/homepage.spec.ts --project=chromium`
- **Before `/gsd:verify-work`:** Full suite must be green
- **Max feedback latency:** 30 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 03-01-01 | 01 | 1 | RT-01 | build | `cd frontend && npm run build` | ✅ | ⬜ pending |
| 03-01-02 | 01 | 1 | RT-02 | build | `cd frontend && npm run build` | ✅ | ⬜ pending |
| 03-02-01 | 02 | 2 | RT-01 | build | `cd frontend && npm run build` | ✅ | ⬜ pending |
| 03-02-02 | 02 | 2 | RT-01 | smoke | `cd frontend && npx playwright test tests/homepage.spec.ts --project=chromium` | ✅ | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [ ] `@portabletext/react` npm install — required before renderer can be written
- [ ] `@tailwindcss/typography` npm install — required for prose baseline per D-04
- [ ] `@plugin "@tailwindcss/typography"` added to globals.css — activates prose in TW v4

*Existing infrastructure (Playwright 1.58.2) covers smoke test needs. No new test framework required.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| RT-01 visual correctness (headings, lists, links render with brand styling) | RT-01 | Requires Sanity content with PT blocks seeded in dev; Playwright smoke tests check page loads but not specific PT class output | Run `npm run dev`, navigate to pages with info sections/image-with-text, visually confirm h2/h3 have `text-brand-red font-serif`, lists have `list-disc`, links open correctly |
| Link `target="_blank"` behavior | RT-01 | Requires content with link marks where `blank=true` is set in Sanity | Create/find a PT block with a link annotation where blank=true, verify new tab behavior |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 30s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
