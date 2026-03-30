---
phase: 5
slug: cache-revalidation
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-27
---

# Phase 5 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Playwright 1.58.2 |
| **Config file** | `frontend/playwright.config.ts` |
| **Quick run command** | `cd frontend && npm run build` |
| **Full suite command** | `cd frontend && npm run build && npm run lint` |
| **Estimated runtime** | ~30 seconds |

---

## Sampling Rate

- **After every task commit:** Run `cd frontend && npm run build`
- **After every plan wave:** Run `cd frontend && npm run build && npm run lint`
- **Before `/gsd:verify-work`:** Full suite must be green + manual end-to-end test per success criteria
- **Max feedback latency:** 30 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 05-01-01 | 01 | 1 | INFRA-05 | Smoke | `cd frontend && npm run build` | ✅ | ⬜ pending |
| 05-01-02 | 01 | 1 | INFRA-05 | Smoke | `cd frontend && npm run build` | ✅ | ⬜ pending |
| 05-01-03 | 01 | 1 | INFRA-06 | Smoke | `cd frontend && npm run build` | ✅ | ⬜ pending |
| 05-01-04 | 01 | 1 | INFRA-05, INFRA-06 | Manual | curl + Vercel function logs | ❌ | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

*Existing infrastructure covers all phase requirements. No new test framework or stubs needed. Validation relies on TypeScript build checks (automated) and manual curl/browser verification (manual).*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Webhook POST triggers revalidateTag for correct type | INFRA-05 | Requires live Sanity → Vercel webhook delivery | 1. Publish a document in Sanity Studio. 2. Check Vercel function logs for `[revalidate] tag=...`. 3. Hard refresh the corresponding page — content should reflect the change. |
| Route handler returns 401 for invalid signature | INFRA-06 | Requires running server to test HTTP responses | `curl -X POST http://localhost:3000/api/revalidate -H "Content-Type: application/json" -d '{"_type":"homePage"}' -w "\n%{http_code}"` — expect 401 |
| Route handler returns 401 for missing signature header | INFRA-06 | Requires running server | Same curl without signature header — expect 401 |
| Route handler returns 400 for missing _type | INFRA-06 | Requires running server with valid signature | Send signed request with empty body — expect 400 |
| Updated content visible after webhook fire | INFRA-05 | End-to-end browser check against live deploy | Publish in Studio → wait 5s → hard refresh page → verify new content |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 30s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
