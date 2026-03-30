---
phase: 4
slug: forms-and-email
status: draft
nyquist_compliant: true
wave_0_complete: true
created: 2026-03-26
---

# Phase 4 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Inline `node -e` content checks (no vitest suite for this phase) |
| **Config file** | N/A — verification is inline per task |
| **Quick run command** | Per-task `<automated>` blocks in each PLAN.md |
| **Full suite command** | `cd frontend && npx tsc --noEmit` |
| **Estimated runtime** | ~15 seconds |

---

## Sampling Rate

- **After every task commit:** Run the task's `<automated>` verify block
- **After every plan wave:** Run `cd frontend && npx tsc --noEmit`
- **Before `/gsd:verify-work`:** `npx tsc --noEmit` and `npm run build` must be green
- **Max feedback latency:** 15 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | Status |
|---------|------|------|-------------|-----------|-------------------|--------|
| 04-01-01 | 01 | 1 | FORM-04 | inline content check | `node -e` checks resend install, form-client.ts content, .env.local.example | ⬜ pending |
| 04-01-02 | 01 | 1 | FORM-03 | inline content check | `node -e` checks email.ts exports, templates, escapeHtml, Promise.allSettled | ⬜ pending |
| 04-02-01 | 02 | 2 | FORM-01, FORM-02 | inline content check | `node -e` checks form.ts 'use server', actions, fieldErrors, formClient.create | ⬜ pending |
| 04-02-02 | 02 | 2 | FORM-01, FORM-02 | inline content check + tsc | `node -e` checks component wiring, button copy, fieldErrors hydration + `npx tsc --noEmit` | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

Existing inline `node -e` content checks in each task's `<automated>` block cover all phase requirements. No separate vitest test stubs needed — the phase creates server-only modules (form-client, email, Server Actions) that depend on Sanity tokens and Resend API keys unavailable in CI. Inline content verification confirms correct file structure, exports, and wiring without requiring runtime credentials.

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Email delivery arrives in inbox | FORM-03 | Requires actual Resend API key and email delivery | Submit form in dev, check email inbox within 30s |
| Sanity Studio shows new document | FORM-01, FORM-02 | Requires Sanity Studio UI interaction | Submit form, open Studio, verify document appears in collection |

---

## Validation Sign-Off

- [x] All tasks have `<automated>` verify blocks with inline content checks
- [x] Sampling continuity: every task has automated verify
- [x] Wave 0 covered: inline checks substitute for vitest stubs (credentials-dependent modules)
- [x] No watch-mode flags
- [x] Feedback latency < 30s
- [x] `nyquist_compliant: true` set in frontmatter

**Approval:** ready
