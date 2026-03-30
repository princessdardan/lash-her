# Phase 4: Forms and Email - Research

**Researched:** 2026-03-26
**Domain:** Next.js Server Actions, Sanity write client, Resend email SDK
**Confidence:** HIGH

---

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

- **D-01:** Single shared action file at `frontend/src/app/actions/form.ts` exporting `submitGeneralInquiry()` and `submitTrainingContact()`. Shared helpers (Sanity write, email send) as private functions within the same file.
- **D-02:** Actions return typed result objects `{ success: boolean; error?: string }`. Form components call the action directly and handle the result. No `useActionState` integration.
- **D-03:** Server-side validation in every action — re-validate using the same validation rules as the client. Defense in depth. Return field-level errors if invalid.
- **D-04:** Exact port of all 4 HTML templates (admin + user variants for both forms) from `backend/src/services/email.ts` into `frontend/src/lib/email.ts`. Same HTML, same inline styles, same `escapeHtml` logic. Zero regression in email appearance.
- **D-05:** Plain exported functions — `sendFormEmails()`, `sendAdminNotification()`, `sendUserConfirmation()` as standalone functions. No class, no singleton. Stateless for Server Action calls.
- **D-06:** If Sanity write succeeds but emails fail: show success to user. Email failure logged server-side but invisible to the user. Matches current Strapi non-blocking pattern.
- **D-07:** If Sanity write fails: return generic error ('Something went wrong, please try again'). Don't send emails. Log the Sanity error server-side. No internal details exposed to the user.
- **D-08:** Dedicated `SANITY_FORM_TOKEN` env var with create-only permissions scoped to `contactForm` and `generalInquiry` document types. Separate from the general `SANITY_WRITE_TOKEN`. Principle of least privilege.
- **D-09:** Separate form client module at `frontend/src/sanity/lib/form-client.ts` using `SANITY_FORM_TOKEN`. Isolated from the general write client used by migration scripts.

### Claude's Discretion

- Internal Server Action implementation details (how to structure the Sanity mutation call)
- Exact server-side validation implementation (reuse form-validation.ts or inline)
- Resend client initialization pattern within the email module
- Environment variable naming for Resend config (RESEND_API_KEY, FROM_EMAIL, ADMIN_EMAIL)
- Whether to add rate limiting or spam protection (not in requirements but reasonable to consider)

### Deferred Ideas (OUT OF SCOPE)

None — discussion stayed within phase scope.
</user_constraints>

---

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| FORM-01 | General inquiry form submits via Next.js Server Action → creates Sanity document + sends Resend admin notification and user confirmation emails in parallel | Server Actions pattern, Sanity client.create(), Resend emails.send() |
| FORM-02 | Training contact form submits via Next.js Server Action → creates Sanity document + sends Resend admin notification and user confirmation emails in parallel | Same as FORM-01, different schema shape |
| FORM-03 | Resend email service moved from `backend/src/services/email.ts` to `frontend/src/lib/email.ts` with equivalent HTML templates | Backend source is fully readable, 4 templates ready to port |
| FORM-04 | Write token scoped to create-only permissions on form submission document types | See critical finding: Sanity token scoping limitation |
</phase_requirements>

---

## Summary

Phase 4 replaces Strapi lifecycle hook-triggered emails with Next.js Server Actions that write directly to Sanity and call Resend. All moving parts are well-understood: the existing codebase provides the form components (with TODO stubs), the email templates (ready to port), the Sanity schema (already defined), and an existing write client pattern to follow.

The core implementation involves three new files: `frontend/src/app/actions/form.ts` (Server Actions), `frontend/src/lib/email.ts` (ported email module), and `frontend/src/sanity/lib/form-client.ts` (write client using `SANITY_FORM_TOKEN`). Both form components (`GeneralInquiryForm`, `ContactFormLabels`) have explicit `// TODO: Phase 4` stubs in their `handleSubmit` functions that are replaced with direct async calls to the Server Actions.

**Critical finding on D-08 (token scoping):** Sanity does NOT support create-only permissions scoped to specific document types on the free tier. Custom resource-based permissions are an Enterprise plan feature only. The `SANITY_FORM_TOKEN` must use the `editor` role, which grants full read+write access. The "principle of least privilege" intent of D-08 is achieved architecturally (separate token for forms) but not at the Sanity permission layer. The planner must document this in the task for token creation so the implementer creates an `editor`-role token named for its form purpose.

**Primary recommendation:** Use `client.create()` with a dedicated editor-role form token. Separate token is still valuable isolation — if the form token leaks, migration scripts using `SANITY_WRITE_TOKEN` are unaffected. Resend `resend` package must be added to `frontend/package.json` (currently only in `backend/`).

---

## Standard Stack

### Core

| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| `resend` | 6.9.4 (published 2026-03-16) | Send transactional emails | Official SDK for Resend API, already used in backend |
| `@sanity/client` | 7.20.0 (already installed) | Write form documents to Sanity | Already present in frontend, provides `client.create()` |
| `server-only` | 0.0.1 (already installed) | Prevent client-side import of server modules | Already used in `write-client.ts`, pattern established |
| `next` | 16.2.1 (already installed) | Server Actions runtime | `'use server'` directive available in app directory |

### Supporting

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| `form-validation.ts` (project) | N/A (existing) | Re-use server-side validation | Import `validateField`, `validateForm` in Server Action for D-03 |

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| HTML string templates | React Email / @react-email | React Email is cleaner but adds dependency; D-04 locks exact HTML port, so React Email is out of scope |
| `validateField` reuse | Zod | Zod not installed; reusing existing `form-validation.ts` avoids new dependency and satisfies D-03 |
| `useActionState` | Manual async call | D-02 locks the pattern to manual call + result handling |

**Installation (new to frontend only):**

```bash
cd frontend && npm install resend
```

**Version verification:** `resend` 6.9.4 verified on 2026-03-26 via `npm view resend dist-tags.latest`. `@sanity/client` 7.20.0 already installed in frontend.

---

## Architecture Patterns

### Recommended Project Structure (new files only)

```
frontend/src/
├── app/
│   └── actions/
│       └── form.ts           # 'use server' — submitGeneralInquiry(), submitTrainingContact()
├── lib/
│   └── email.ts              # sendFormEmails(), sendAdminNotification(), sendUserConfirmation()
└── sanity/
    └── lib/
        └── form-client.ts    # write client using SANITY_FORM_TOKEN
```

### Pattern 1: Server Action (D-01, D-02, D-03)

The `'use server'` directive at the top of a **file** marks all exports as Server Actions. The action is called directly from the client component's event handler as an async function — no form `action=` attribute needed.

```typescript
// frontend/src/app/actions/form.ts
'use server'

import { validateForm } from '@/lib/form-validation'
import { formClient } from '@/sanity/lib/form-client'
import { sendFormEmails } from '@/lib/email'

export interface FormActionResult {
  success: boolean
  error?: string
}

export async function submitGeneralInquiry(
  data: GeneralInquiryData
): Promise<FormActionResult> {
  // D-03: Server-side re-validation
  const { isValid, errors } = validateForm(
    data as Record<string, string>,
    GENERAL_INQUIRY_VALIDATION_CONFIG
  )
  if (!isValid) {
    return { success: false, error: 'Please fix the form errors and try again.' }
  }

  // D-07: Sanity write — error blocks email send
  try {
    await formClient.create({
      _type: 'generalInquiry',
      name: data.name,
      email: data.email,
      phone: data.phone ?? undefined,
      instagram: data.instagram ?? undefined,
      message: data.message,
    })
  } catch (err) {
    console.error('[submitGeneralInquiry] Sanity write failed:', err)
    return { success: false, error: 'Something went wrong, please try again.' }
  }

  // D-06: Email failure is non-blocking — user sees success regardless
  try {
    await sendFormEmails('general-inquiry', data)
  } catch (err) {
    console.error('[submitGeneralInquiry] Email send failed:', err)
  }

  return { success: true }
}
```

**Calling the action from the client component** (replaces the TODO stub in `handleSubmit`):

```typescript
// Inside GeneralInquiryForm.handleSubmit — replaces the TODO block
setIsSubmitting(true)
setSubmitStatus({ type: null, message: '' })

const result = await submitGeneralInquiry(formData)

if (result.success) {
  setSubmitStatus({ type: 'success', message: 'Thank you! Your submission has been received.' })
  // reset form state...
} else {
  setSubmitStatus({ type: 'error', message: result.error ?? 'Something went wrong, please try again.' })
}
setIsSubmitting(false)
```

### Pattern 2: Form Client (D-09)

Follows the exact same pattern as `write-client.ts` but uses a different env var:

```typescript
// frontend/src/sanity/lib/form-client.ts
import 'server-only'
import { createClient } from '@sanity/client'
import { apiVersion, dataset, projectId } from '../env'

export const formClient = createClient({
  projectId,
  dataset,
  apiVersion,
  useCdn: false,
  token: process.env.SANITY_FORM_TOKEN,
})
```

### Pattern 3: Email Module (D-04, D-05)

The backend `EmailService` class becomes stateless exported functions. Resend client is initialized at module level with a lazy-init guard pattern so missing env vars don't crash the module import:

```typescript
// frontend/src/lib/email.ts
import { Resend } from 'resend'

// Source: Resend Node.js SDK docs — https://resend.com/docs/send-with-nodejs
// Response shape: { data: { id: string } | null, error: { message, name } | null }
const resend = new Resend(process.env.RESEND_API_KEY)

export async function sendFormEmails(
  formType: 'general-inquiry' | 'training-contact',
  formData: GeneralInquiryData | TrainingContactData
): Promise<void> {
  await Promise.allSettled([
    sendAdminNotification(formType, formData),
    sendUserConfirmation(formType, formData),
  ])
}

async function sendAdminNotification(...): Promise<void> {
  const { error } = await resend.emails.send({
    from: process.env.FROM_EMAIL!,
    to: process.env.ADMIN_EMAIL!,
    subject: getAdminSubject(formType, formData),
    html: getAdminEmailHtml(formType, formData),
  })
  if (error) throw new Error(error.message)
}
```

### Pattern 4: Sanity `client.create()` shape

```typescript
// Source: @sanity/client docs, verified via WebSearch 2026-03-26
await formClient.create({
  _type: 'contactForm',          // required — matches schema document type
  name: data.name,               // required per schema validation
  email: data.email,             // required per schema validation
  phone: data.phone,             // required for training form
  location: data.location,       // required for training form
  instagram: data.instagram,     // required for training form
  experience: data.experience,   // required for training form
  interest: data.interest,       // required for training form
  clients: data.clients          // optional number
  info: data.info,               // optional text
})
// Returns: SanityDocument with _id, _rev, _createdAt etc.
```

Note: `liveEdit: true` is set on both schemas — documents are published immediately on create, no draft phase. This is correct for form submissions.

### Anti-Patterns to Avoid

- **Import `formClient` in client components:** The form client uses `server-only` — importing it in a `'use client'` component causes a build error. The Server Action is the only caller.
- **Exposing Sanity error details to the user:** D-07 requires a generic error message. Don't propagate `err.message` from the Sanity client to the return value.
- **Awaiting emails serially:** Use `Promise.allSettled` to send admin notification and user confirmation in parallel, matching the existing backend pattern.
- **Constructing Resend client inside sendFormEmails:** Initialize once at module level — the client is stateless and re-creating it per call wastes memory.
- **Skipping the `import 'server-only'` guard:** Without it, if the module is accidentally imported client-side (e.g., via a bad import path), it will expose `SANITY_FORM_TOKEN` to the browser bundle.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Sending email | Custom SMTP/nodemailer | `resend` SDK | Already in backend, API key already acquired, deliverability handled |
| HTML email XSS protection | Custom sanitizer | Port `escapeHtml()` from `backend/src/services/email.ts` (5 chars covered) | D-04 requires exact port — the existing function is correct |
| Server-side form validation | Custom validators | Import `validateField`, `validateForm` from `@/lib/form-validation` | D-03 uses existing rules — no new dependency needed |

**Key insight:** Everything needed already exists in the repo. This phase is primarily wiring, not building new infrastructure.

---

## Common Pitfalls

### Pitfall 1: Token Scoping Mismatch with D-08

**What goes wrong:** D-08 specifies "create-only permissions scoped to `contactForm` and `generalInquiry` document types." This level of granularity requires Sanity Enterprise plan.
**Why it happens:** Sanity free tier supports only `viewer`, `editor`, and `deploy-studio` token roles. Custom resource-based permissions (filter by `_type`) are paid-only.
**How to avoid:** Create a dedicated `editor`-role token named `FORM_WRITE` (or similar descriptive name) in Sanity Manage. Document that the token has full read+write access but is used exclusively for form creation. The isolation intent of D-08 is still achieved — the token is separate from `SANITY_WRITE_TOKEN` used by migration scripts.
**Warning signs:** If you see "Insufficient permissions" errors when calling `client.create()` on the free tier, verify the token role is `editor`, not `viewer`.

### Pitfall 2: `resend` Package Not Installed in Frontend

**What goes wrong:** `resend` is only in `backend/package.json` — it is not in `frontend/package.json`. Importing it in `frontend/src/lib/email.ts` will fail at build time.
**Why it happens:** The email service was entirely in the Strapi backend. No one added it to the frontend yet.
**How to avoid:** `npm install resend` in the `frontend/` directory as the first task of this phase.
**Warning signs:** `Cannot find module 'resend'` during `next build` or `next dev`.

### Pitfall 3: `handleSubmit` is Synchronous — Must Be Made Async

**What goes wrong:** Both form components declare `handleSubmit` as `(e: React.FormEvent) => void`. Calling `await submitGeneralInquiry(...)` inside a synchronous function silently drops the result.
**Why it happens:** The original stub set state synchronously — no async needed. Now it needs to await the Server Action.
**How to avoid:** Change signature to `async (e: React.FormEvent) => Promise<void>`. React event handlers accept async functions without issue.
**Warning signs:** `isSubmitting` stays `false` before the action resolves, form shows no loading state.

### Pitfall 4: `FormData` Type Name Collision

**What goes wrong:** `contact-components.tsx` locally defines `type FormData = { name: string; ... }`. The Resend/Server Action file also deals with form data objects. The local `FormData` type shadows the browser's global `FormData` Web API.
**Why it happens:** Local `type FormData` at the top of the component file collides with the global Web API type if someone tries to import it.
**How to avoid:** In the Server Action file, use distinct interface names (`GeneralInquiryData`, `TrainingContactData`) that match the existing type definitions in `backend/src/services/email.ts`. Do not name the type `FormData`.

### Pitfall 5: Resend `{ data, error }` Response Pattern vs. Try/Catch

**What goes wrong:** Resend SDK v3+ uses a `{ data, error }` response shape instead of throwing. Wrapping `resend.emails.send()` in try/catch without also checking `error` means API errors are silently swallowed.
**Why it happens:** The SDK deliberately doesn't throw on API errors — it returns `{ data: null, error: { message, name } }`.
**How to avoid:** Check both: `const { data, error } = await resend.emails.send(...)`. If `error`, throw or log. See code examples below.

### Pitfall 6: Missing `'use server'` Directive Position

**What goes wrong:** Putting `'use server'` inside the function body (inline Server Action pattern) instead of at the top of the file means the file exports are not all Server Actions. This works for inline Server Actions inside Server Components, but since these actions are in a standalone file, the directive must be at the top of the file.
**Why it happens:** The Next.js docs show both patterns; inline is for Server Components, file-top is for separate action modules.
**How to avoid:** `frontend/src/app/actions/form.ts` must start with `'use server'` as its first statement (before imports). This is the module-level directive.

---

## Code Examples

Verified patterns from official sources:

### Resend `emails.send()` — Correct Response Handling

```typescript
// Source: https://resend.com/docs/send-with-nodejs (verified 2026-03-26)
const resend = new Resend(process.env.RESEND_API_KEY)

const { data, error } = await resend.emails.send({
  from: 'Lash Her <noreply@lashher.com>',
  to: 'recipient@example.com',
  subject: 'Subject Line',
  html: '<p>HTML content</p>',
})

if (error) {
  // error.message and error.name available
  throw new Error(`Resend API error: ${error.message}`)
}
// data.id is the sent email ID if needed for logging
```

### Sanity `client.create()` — Basic Document Creation

```typescript
// Source: GitHub sanity-io/client README (verified 2026-03-26)
// Return type: Promise<SanityDocument>
const created = await formClient.create({
  _type: 'generalInquiry',
  name: 'Test User',
  email: 'test@example.com',
  message: 'Hello',
})
// created._id, created._rev, created._createdAt available
```

### Server Action Module Pattern

```typescript
// Source: https://nextjs.org/docs/app/guides/forms (Next.js 16.2.1 docs, 2026-03-25)
// File must start with 'use server' for all exports to be Server Actions
'use server'

export async function myAction(data: MyData): Promise<{ success: boolean; error?: string }> {
  // runs on the server — can access env vars, write to DB
  // return serializable plain object — no class instances, no functions
}
```

### Calling Server Action from Client Component

```typescript
// Source: Next.js docs — calling Server Functions from event handlers
'use client'
import { myAction } from '@/app/actions/form'

// Inside component:
const handleSubmit = async (e: React.FormEvent) => {
  e.preventDefault()
  // Client-side validation first...
  const result = await myAction(formData)
  // Handle result
}
```

---

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Strapi lifecycle `afterCreate` hook triggers email | Next.js Server Action calls Resend directly | Phase 4 | Eliminates Strapi dependency for form email |
| `EmailService` class singleton (backend) | Stateless exported functions (frontend) | Phase 4 | Simpler, works as pure server-side module |
| `resend.emails.send()` throws on error | Returns `{ data, error }` pattern | Resend SDK v3+ | Must check `error` property, not rely on try/catch alone |

**Deprecated/outdated:**
- `backend/src/services/email.ts`: EmailService singleton — replaced by `frontend/src/lib/email.ts` standalone functions per D-05
- `backend/src/api/*/lifecycles.ts`: Strapi lifecycle hooks — replaced by Server Actions per D-01

---

## Open Questions

1. **SANITY_FORM_TOKEN token role on free tier**
   - What we know: Custom document-type scoping requires Enterprise plan. Free tier supports `editor` (full read+write) or `viewer` (read-only).
   - What's unclear: Whether the planner should document this constraint clearly or accept "editor role, form-only use" as satisfying the spirit of D-08.
   - Recommendation: Accept `editor` role as the implementation of D-08. Document in the task that this token must be created with `--role=editor` and used ONLY for form submissions. No code change required — the separation of tokens still provides isolation.

2. **`.env.local` update for form env vars**
   - What we know: `.env.local.example` does not yet contain `SANITY_FORM_TOKEN`, `RESEND_API_KEY`, `FROM_EMAIL`, or `ADMIN_EMAIL`.
   - What's unclear: Whether updating `.env.local.example` is in scope or deferred to Phase 5 (infrastructure).
   - Recommendation: Include it as a documentation step in Phase 4 since the form actions directly require these vars to function.

---

## Environment Availability

| Dependency | Required By | Available | Version | Fallback |
|------------|------------|-----------|---------|----------|
| Node.js | Server Actions runtime | ✓ | Built into Next.js | — |
| `resend` npm package | `frontend/src/lib/email.ts` | ✗ (not in frontend yet) | 6.9.4 available | No fallback — must install |
| `@sanity/client` | `form-client.ts` | ✓ | 7.20.0 installed | — |
| `server-only` | Server module guard | ✓ | 0.0.1 installed | — |
| `RESEND_API_KEY` env var | Resend initialization | Unknown (in backend `.env`) | — | Phase fails without it |
| `SANITY_FORM_TOKEN` env var | `form-client.ts` | ✗ (not created yet) | — | Phase fails without it |
| `FROM_EMAIL` env var | `sendAdminNotification()` | Unknown (in backend `.env`) | — | Phase fails without it |
| `ADMIN_EMAIL` env var | `sendAdminNotification()` | Unknown (in backend `.env`) | — | Phase fails without it |

**Missing dependencies with no fallback:**
- `resend` package in `frontend/` — must `npm install resend` before writing `email.ts`
- `SANITY_FORM_TOKEN` — must be created via Sanity Manage dashboard (editor role) before testing
- `RESEND_API_KEY`, `FROM_EMAIL`, `ADMIN_EMAIL` — exist in backend `.env`; must be added to `frontend/.env.local` for local dev and to Vercel env vars for production

**Missing dependencies with fallback:**
- None. All blockers have clear remediation steps.

---

## Validation Architecture

### Test Framework

| Property | Value |
|----------|-------|
| Framework | Playwright 1.58.2 |
| Config file | `frontend/playwright.config.ts` |
| Quick run command | `cd frontend && npm test -- tests/contact.spec.ts` |
| Full suite command | `cd frontend && npm test` |

### Phase Requirements → Test Map

| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| FORM-01 | General inquiry form submits successfully, creates Sanity document, triggers emails | E2E (requires live Sanity + Resend keys) | `cd frontend && npm test -- tests/contact.spec.ts` | ✅ (partial — existing test has TODO stubs) |
| FORM-02 | Training contact form submits successfully, creates Sanity document, triggers emails | E2E (requires live Sanity + Resend keys) | `cd frontend && npm test -- tests/training.spec.ts` | ✅ (partial — `training.spec.ts` exists) |
| FORM-03 | Email module is server-side only, all 4 templates rendered correctly | Manual (email preview) + build check | `cd frontend && npm run build` | ❌ Wave 0 gap |
| FORM-04 | Form token is separate from write token; Sanity Studio shows created documents | Manual (studio inspection) | Manual verification in `/studio` | Manual-only |

**Manual-only justifications:**
- FORM-03 template correctness requires visual email preview — no automated assertion possible without email testing service (not in scope)
- FORM-04 requires human to verify Sanity Studio shows form submissions — Studio UI cannot be driven by Playwright in this setup

### Sampling Rate

- **Per task commit:** `cd frontend && npm run build` (confirms no TypeScript or import errors)
- **Per wave merge:** `cd frontend && npm test -- tests/contact.spec.ts tests/training.spec.ts`
- **Phase gate:** Full suite green before `/gsd:verify-work`

### Wave 0 Gaps

- [ ] `frontend/src/app/actions/form.ts` — does not exist yet (the file to create)
- [ ] `frontend/src/lib/email.ts` — does not exist yet (the file to create)
- [ ] `frontend/src/sanity/lib/form-client.ts` — does not exist yet (the file to create)
- No test infrastructure gaps — Playwright and existing spec files are sufficient; `contact.spec.ts` and `training.spec.ts` already exercise form submission flows

---

## Project Constraints (from CLAUDE.md)

No project-level `CLAUDE.md` found at `/Users/dardan/Documents/lash-her/CLAUDE.md`.

Global `~/.claude/CLAUDE.md` applies:
- **TypeScript-strict:** Explicit types on function signatures, no `any`, strict null checks
- **Named exports over default exports** — `submitGeneralInquiry`, `submitTrainingContact`, `sendFormEmails` all named exports
- **`interface` for object shapes** — use `interface GeneralInquiryData` not `type`
- **Early returns** — return `{ success: false, error: '...' }` before the happy path
- **Typed errors** — never `catch (e: any)`, use `e instanceof Error ? e.message : String(e)`
- **Package manager:** `npm` (not pnpm or yarn)

---

## Sources

### Primary (HIGH confidence)
- Next.js 16.2.1 official docs — https://nextjs.org/docs/app/guides/forms — Server Actions form pattern, verified 2026-03-25
- Resend official docs — https://resend.com/docs/send-with-nodejs — `emails.send()` API, `{ data, error }` response shape, verified 2026-03-26
- `@sanity/client` GitHub README — https://github.com/sanity-io/client — `client.create()` signature and return type, verified 2026-03-26
- Project source — `backend/src/services/email.ts` — All 4 HTML templates, `escapeHtml` utility, type definitions for `GeneralInquiryData` / `TrainingContactData`
- Project source — `frontend/src/sanity/lib/write-client.ts` — Established `createClient()` pattern for form-client.ts
- Project source — `frontend/src/lib/form-validation.ts` — `validateField`, `validateForm`, `FieldValidationConfig` — reusable server-side

### Secondary (MEDIUM confidence)
- Sanity roles docs — https://www.sanity.io/docs/content-lake/roles-concepts — Token role capabilities and free tier limitations (custom scoping = Enterprise only)
- Sanity community answer — https://www.sanity.io/answers/changes-to-sanity-s-api-token-permissions-on-the-free-plan -- — Confirmed `editor` role available on free plan

### Tertiary (LOW confidence)
- None.

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — `resend` verified via npm registry, `@sanity/client` already installed and in use
- Architecture: HIGH — patterns copied from existing project conventions + official Next.js and Resend docs
- Pitfalls: HIGH — token scoping limitation verified against Sanity docs; `resend` installation gap confirmed by checking `frontend/node_modules/`; async `handleSubmit` gap confirmed by reading the component source

**Research date:** 2026-03-26
**Valid until:** 2026-04-25 (stable APIs — Next.js, Resend, Sanity client are unlikely to break patterns within 30 days)
