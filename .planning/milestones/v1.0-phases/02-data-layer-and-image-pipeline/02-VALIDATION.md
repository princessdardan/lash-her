---
phase: 02
slug: data-layer-and-image-pipeline
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-25
---

# Phase 02 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Playwright 1.58.2 |
| **Config file** | `frontend/playwright.config.ts` |
| **Quick run command** | `cd frontend && npx tsc --noEmit` |
| **Full suite command** | `cd frontend && npx playwright test` |
| **Estimated runtime** | ~60 seconds (tsc ~15s, Playwright ~45s) |

---

## Sampling Rate

- **After every task commit:** Run `cd frontend && npx tsc --noEmit`
- **After every plan wave:** Run `cd frontend && npx playwright test --project=chromium`
- **Before `/gsd:verify-work`:** Full suite must be green
- **Max feedback latency:** 15 seconds (tsc check)

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 02-01-01 | 01 | 1 | DATA-01 | E2E smoke | `npx playwright test --project=chromium` | Yes | ⬜ pending |
| 02-01-02 | 01 | 1 | DATA-05 | E2E smoke | `npx playwright test --project=chromium` | Yes | ⬜ pending |
| 02-02-01 | 02 | 1 | DATA-02 | E2E visual | `npx playwright test homepage.spec.ts gallery.spec.ts training.spec.ts` | Yes | ⬜ pending |
| 02-02-02 | 02 | 1 | DATA-03 | Build check | `cd frontend && npx tsc --noEmit` | n/a | ⬜ pending |
| 02-03-01 | 03 | 2 | IMG-01 | E2E visual | `npx playwright test homepage.spec.ts` | Yes (partial) | ⬜ pending |
| 02-03-02 | 03 | 2 | IMG-02 | E2E network | Manual observation or playwright network intercept | Yes (partial) | ⬜ pending |
| 02-03-03 | 03 | 2 | IMG-03 | Build/runtime | `cd frontend && npx next build 2>&1 \| grep -i "hostname"` | n/a | ⬜ pending |
| 02-03-04 | 03 | 2 | IMG-04 | Manual | Check browser devtools Network tab | Manual | ⬜ pending |
| 02-04-01 | 04 | 2 | DATA-04 | E2E smoke | `npx playwright test --project=chromium training.spec.ts` | Yes | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [ ] Update Playwright test mocks — existing `setupApiMocks(page)` intercepts Strapi API calls, needs updating/disabling for Sanity GROQ endpoints
- [ ] Verify Sanity staging dataset has seeded content — all E2E tests require data to exist

*If neither: "Existing infrastructure covers all phase requirements."*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Image transforms (width/height/format) | IMG-04 | Requires browser devtools to verify CDN query params | Open page in browser, inspect Network tab for cdn.sanity.io requests, verify `?w=`, `?h=`, `?fm=` params |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 15s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
