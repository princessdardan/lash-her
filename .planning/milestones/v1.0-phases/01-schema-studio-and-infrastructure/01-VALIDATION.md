---
phase: 1
slug: schema-studio-and-infrastructure
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-24
---

# Phase 1 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Playwright (existing E2E) + manual GROQ queries |
| **Config file** | `frontend/playwright.config.ts` |
| **Quick run command** | `cd frontend && npx playwright test --grep "studio"` |
| **Full suite command** | `cd frontend && npx playwright test` |
| **Estimated runtime** | ~30 seconds |

---

## Sampling Rate

- **After every task commit:** Run TypeScript compilation check `cd frontend && npx tsc --noEmit`
- **After every plan wave:** Run full type check + verify Studio loads
- **Before `/gsd:verify-work`:** Full suite must be green
- **Max feedback latency:** 30 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| TBD | TBD | TBD | SCHEMA-01 | type-check | `npx tsc --noEmit` | TBD | ⬜ pending |
| TBD | TBD | TBD | SCHEMA-02 | type-check | `npx tsc --noEmit` | TBD | ⬜ pending |
| TBD | TBD | TBD | SCHEMA-03 | type-check | `npx tsc --noEmit` | TBD | ⬜ pending |
| TBD | TBD | TBD | SCHEMA-04 | type-check | `npx tsc --noEmit` | TBD | ⬜ pending |
| TBD | TBD | TBD | SCHEMA-05 | type-check | `npx tsc --noEmit` | TBD | ⬜ pending |
| TBD | TBD | TBD | INFRA-01 | manual | Studio loads at /studio | TBD | ⬜ pending |
| TBD | TBD | TBD | INFRA-02 | manual | GROQ query returns data | TBD | ⬜ pending |
| TBD | TBD | TBD | INFRA-03 | type-check | `npx tsc --noEmit` | TBD | ⬜ pending |
| TBD | TBD | TBD | INFRA-04 | type-check | `npx tsc --noEmit` | TBD | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [ ] Sanity project created and dataset provisioned
- [ ] Environment variables set in `frontend/.env.local`
- [ ] `sanity`, `next-sanity`, `@sanity/client` packages installed

*Existing Playwright infrastructure covers E2E verification needs.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Studio loads at /studio | INFRA-01 | Requires running dev server + browser | Navigate to localhost:3000/studio, verify sidebar shows all document types |
| Singleton click opens editor | SCHEMA-01 | Visual Studio interaction | Click "Homepage" in sidebar, verify editor opens without list view |
| Block type list in array field | SCHEMA-02 | Visual Studio interaction | Edit Homepage, click + in blocks array, verify all layout types appear |
| GROQ query returns data | INFRA-02 | Requires Sanity API call | Run `sanity documents query '*[_type == "homePage"]'` against staging dataset |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 30s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
