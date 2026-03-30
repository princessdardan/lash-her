---
phase: 04-forms-and-email
plan: 01
subsystem: api
tags: [resend, email, sanity, typescript, html-templates]

# Dependency graph
requires:
  - phase: 01-schema-studio-and-infrastructure
    provides: Sanity env.ts with projectId/dataset/apiVersion exports
  - phase: 02-data-layer-and-image-pipeline
    provides: write-client.ts pattern for form-client to follow
provides:
  - Resend package installed in frontend (^6.9.4)
  - formClient: Sanity write client using SANITY_FORM_TOKEN for form submissions
  - sendFormEmails: stateless function for parallel email sending
  - sendAdminNotification: admin notification with HTML template dispatch
  - sendUserConfirmation: user confirmation with HTML template dispatch
  - 4 HTML email templates identical to backend
  - .env.local.example updated with all required env vars
affects: [04-02-server-actions, plan-02-contact-forms, plan-02-training-contact]

# Tech tracking
tech-stack:
  added: [resend@6.9.4, postal-mime@2.7.3, svix@1.86.0]
  patterns:
    - "form-client.ts uses import 'server-only' guard to prevent token exposure"
    - "email.ts uses module-level Resend instantiation (not inside function)"
    - "sendFormEmails uses Promise.allSettled (not Promise.all) for parallel sends"
    - "Resend v3+ error pattern: check { error } not try/catch"

key-files:
  created:
    - frontend/src/sanity/lib/form-client.ts
    - frontend/src/lib/email.ts
    - frontend/.env.local.example
  modified:
    - frontend/package.json

key-decisions:
  - "SANITY_FORM_TOKEN uses editor role (free tier has no create-only scoped permissions — least privilege via separate token, not permission layer)"
  - "sendFormEmails does not re-throw from Promise.allSettled — Server Action caller wraps in try/catch per D-06"
  - "Resend SDK v3+ returns { data, error } instead of throwing — must check error property explicitly"
  - "npm install resend blocked by motion-plus placeholder token — installed manually via isolated temp dir + node_modules copy + package-lock.json patch"

patterns-established:
  - "Server-only guard: import 'server-only' as first import in any file using secret env vars"
  - "Named exports only for utility modules — no class, no singleton, no default export"

requirements-completed: [FORM-03, FORM-04]

# Metrics
duration: 14min
completed: 2026-03-27
---

# Phase 4 Plan 01: Forms and Email Infrastructure Summary

**Resend package installed, form-specific Sanity write client with SANITY_FORM_TOKEN, and 4 HTML email templates ported from backend as stateless TypeScript functions**

## Performance

- **Duration:** 14 min
- **Started:** 2026-03-27T18:46:22Z
- **Completed:** 2026-03-27T18:59:47Z
- **Tasks:** 2
- **Files modified:** 4 (3 created, 1 modified)

## Accomplishments
- Resend v6.9.4 installed in frontend with all transitive dependencies (postal-mime, svix, standardwebhooks, uuid, fast-sha256, @stablelib/base64) and package-lock.json updated
- `form-client.ts` created with `import "server-only"` guard, `formClient` export using `SANITY_FORM_TOKEN` — isolated from `SANITY_WRITE_TOKEN` per D-08/D-09
- `email.ts` created with all 4 HTML email templates ported exactly from backend, 607 lines, stateless exported functions per D-05
- `.env.local.example` updated with SANITY_FORM_TOKEN, RESEND_API_KEY, FROM_EMAIL, ADMIN_EMAIL

## Task Commits

Each task was committed atomically:

1. **Task 1: Install resend and create form-specific Sanity write client** - `56a05f2` (feat)
2. **Task 2: Port email module from backend with all 4 HTML templates** - `16fe6b4` (feat)

**Plan metadata:** (docs commit — see state update)

## Files Created/Modified
- `frontend/src/sanity/lib/form-client.ts` - Form-specific Sanity write client using SANITY_FORM_TOKEN with server-only guard
- `frontend/src/lib/email.ts` - Email sending module with 4 HTML templates, escapeHtml utility, Resend client initialization, and parallel send via Promise.allSettled
- `frontend/.env.local.example` - Updated with SANITY_FORM_TOKEN, RESEND_API_KEY, FROM_EMAIL, ADMIN_EMAIL documentation
- `frontend/package.json` - Added resend ^6.9.4 to dependencies

## Decisions Made
- `SANITY_FORM_TOKEN` requires editor role (not create-only) because Sanity free tier doesn't support document-type-scoped permissions. Principle of least privilege is enforced architecturally (separate token) not at the permission layer.
- `sendFormEmails` uses `Promise.allSettled` so both emails are attempted even if one fails. It does NOT re-throw — the Server Action caller handles errors per D-06.
- Resend SDK v3+ returns `{ data, error }` (does not throw on API errors) — `sendAdminNotification` and `sendUserConfirmation` explicitly check the `error` property.

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 3 - Blocking] npm install blocked by motion-plus/motion-studio placeholder tokens**
- **Found during:** Task 1 (Install resend)
- **Issue:** `package.json` references `motion-plus` and `motion-studio` from `api.motion.dev` with placeholder token `__MOTION_DEV_TOKEN__`, causing `npm install` to fail with 401 Unauthorized
- **Fix:** Installed resend in isolated temp directory, copied `node_modules/resend` and all transitive dependencies to main frontend `node_modules`, then manually patched `package-lock.json` via Python to add the resend entries. Updated `package.json` directly.
- **Files modified:** frontend/package.json, frontend/package-lock.json (main project)
- **Verification:** `node -e "require('resend')"` succeeds; `grep '"resend"' package.json` finds entry
- **Committed in:** `56a05f2` (Task 1 commit)

---

**Total deviations:** 1 auto-fixed (1 blocking)
**Impact on plan:** Motion package placeholder token is a pre-existing constraint, not introduced by this plan. Workaround is clean and preserves correct package.json/package-lock.json state.

## Issues Encountered
- TypeScript compilation (`npx tsc --noEmit`) shows pre-existing errors in `strapi-data-api.ts`, `strapi-loaders.ts`, and `error-handler.ts` — these are from Phase 2 Strapi renaming and are not related to the new files. The new `form-client.ts` and `email.ts` produce zero TypeScript errors.

## User Setup Required

Before Plan 02 (Server Actions) can be tested end-to-end, the following external services need configuration:

**Sanity:**
- Create a new API token at manage.sanity.io → Project → API → Tokens → Add token with **Editor** role, name it "Form Submissions"
- Set `SANITY_FORM_TOKEN=<token>` in `.env.local`

**Resend:**
- Create API key at resend.com/api-keys
- Set `RESEND_API_KEY=re_<key>` in `.env.local`
- Set `FROM_EMAIL=noreply@yourdomain.com` (must be a verified sender in Resend)
- Set `ADMIN_EMAIL=admin@yourdomain.com` (receives form submission notifications)

## Next Phase Readiness
- `formClient` and email functions are ready for Server Action consumption in Plan 02
- Both modules are server-side only (form-client uses `server-only` guard, email uses `RESEND_API_KEY`)
- No client-side code changes needed

## Self-Check

**Files exist:**
- `frontend/src/sanity/lib/form-client.ts` — FOUND
- `frontend/src/lib/email.ts` — FOUND
- `frontend/.env.local.example` — FOUND

**Commits exist:**
- `56a05f2` — FOUND (Task 1)
- `16fe6b4` — FOUND (Task 2)

## Self-Check: PASSED

---
*Phase: 04-forms-and-email*
*Completed: 2026-03-27*
