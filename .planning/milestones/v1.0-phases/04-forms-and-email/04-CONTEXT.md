# Phase 4: Forms and Email - Context

**Gathered:** 2026-03-26
**Status:** Ready for planning

<domain>
## Phase Boundary

Replace Strapi lifecycle hooks with Next.js Server Actions that write form submissions to Sanity as documents and send Resend admin notification and user confirmation emails directly — with no dependency on the Strapi backend. Both the general inquiry form and training contact form must submit successfully, create Sanity documents, and trigger emails.

</domain>

<decisions>
## Implementation Decisions

### Server Action Organization
- **D-01:** Single shared action file at `frontend/src/app/actions/form.ts` exporting `submitGeneralInquiry()` and `submitTrainingContact()`. Shared helpers (Sanity write, email send) as private functions within the same file.
- **D-02:** Actions return typed result objects `{ success: boolean; error?: string }`. Form components call the action directly and handle the result. No `useActionState` integration.
- **D-03:** Server-side validation in every action — re-validate using the same validation rules as the client. Defense in depth. Return field-level errors if invalid.

### Email Template Migration
- **D-04:** Exact port of all 4 HTML templates (admin + user variants for both forms) from `backend/src/services/email.ts` into `frontend/src/lib/email.ts`. Same HTML, same inline styles, same `escapeHtml` logic. Zero regression in email appearance.
- **D-05:** Plain exported functions — `sendFormEmails()`, `sendAdminNotification()`, `sendUserConfirmation()` as standalone functions. No class, no singleton. Stateless for Server Action calls.

### Submission Error Handling
- **D-06:** If Sanity write succeeds but emails fail: show success to user. Email failure logged server-side but invisible to the user. Matches current Strapi non-blocking pattern.
- **D-07:** If Sanity write fails: return generic error ('Something went wrong, please try again'). Don't send emails. Log the Sanity error server-side. No internal details exposed to the user.

### Write Token Scoping
- **D-08:** Dedicated `SANITY_FORM_TOKEN` env var with create-only permissions scoped to `contactForm` and `generalInquiry` document types. Separate from the general `SANITY_WRITE_TOKEN`. Principle of least privilege.
- **D-09:** Separate form client module at `frontend/src/sanity/lib/form-client.ts` using `SANITY_FORM_TOKEN`. Isolated from the general write client used by migration scripts.

### Claude's Discretion
- Internal Server Action implementation details (how to structure the Sanity mutation call)
- Exact server-side validation implementation (reuse form-validation.ts or inline)
- Resend client initialization pattern within the email module
- Environment variable naming for Resend config (RESEND_API_KEY, FROM_EMAIL, ADMIN_EMAIL)
- Whether to add rate limiting or spam protection (not in requirements but reasonable to consider)

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Current Backend Email System (being replaced)
- `backend/src/services/email.ts` — EmailService class with all 4 HTML templates, escapeHtml utility, Resend integration, GeneralInquiryData/TrainingContactData type definitions
- `backend/src/api/contact-form/content-types/contact-form/lifecycles.ts` — Training contact afterCreate hook (email trigger pattern)
- `backend/src/api/general-inquiry/content-types/general-inquiry/lifecycles.ts` — General inquiry afterCreate hook (email trigger pattern)

### Frontend Form Components (being wired to Server Actions)
- `frontend/src/components/custom/collection/general-inquiry.tsx` — GeneralInquiryForm with validation, handleSubmit stub marked "TODO: Phase 4"
- `frontend/src/components/custom/collection/contact-components.tsx` — ContactFormLabels with validation, handleSubmit stub marked "TODO: Phase 4"

### Sanity Schemas (target document types)
- `frontend/src/sanity/schemas/documents/contact-form.ts` — contactForm schema with liveEdit: true, all fields defined
- `frontend/src/sanity/schemas/documents/general-inquiry.ts` — generalInquiry schema with liveEdit: true, all fields defined

### Sanity Infrastructure
- `frontend/src/sanity/lib/write-client.ts` — Existing write client (general purpose, not for forms)
- `frontend/src/sanity/env.ts` — Environment variable configuration pattern

### Validation
- `frontend/src/lib/form-validation.ts` — validateField(), validateForm(), FieldValidationConfig, regex patterns

### Project Context
- `.planning/REQUIREMENTS.md` — FORM-01 through FORM-04
- `.planning/ROADMAP.md` — Phase 4 success criteria
- `.planning/phases/01-schema-studio-and-infrastructure/01-CONTEXT.md` — D-12: contactFormLabels/generalInquiryLabels naming; liveEdit: true for form submissions

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- `frontend/src/lib/form-validation.ts` — Complete validation framework with validateField(), validateForm(), regex patterns for email/phone. Can be reused server-side.
- `frontend/src/sanity/lib/write-client.ts` — Write client pattern to follow for form-client.ts
- `frontend/src/sanity/env.ts` — Environment variable access pattern
- `backend/src/services/email.ts` — All 4 HTML email templates ready to port, escapeHtml utility, type definitions for form data

### Established Patterns
- **Client components with `'use client'`** — Both form components are already client components with useState-based form state
- **Validation on blur then change** — Established pattern in both forms using touchedFields tracking
- **Named exports** — All components use named exports
- **`@/` path alias** — Imports use `@/lib/...`, `@/sanity/...`
- **Non-blocking email errors** — Current Strapi pattern: email failures logged but don't prevent form save success

### Integration Points
- `frontend/src/components/custom/collection/general-inquiry.tsx:90-118` — handleSubmit function to replace with Server Action call
- `frontend/src/components/custom/collection/contact-components.tsx:127-158` — handleSubmit function to replace with Server Action call
- `frontend/src/app/actions/` — New directory for Server Actions (no existing actions in project)
- `frontend/src/lib/email.ts` — New email module (replaces backend/src/services/email.ts)
- `frontend/src/sanity/lib/form-client.ts` — New form-specific Sanity client
- `.env.local` — Needs SANITY_FORM_TOKEN, RESEND_API_KEY, FROM_EMAIL, ADMIN_EMAIL

</code_context>

<specifics>
## Specific Ideas

No specific requirements — open to standard Next.js Server Action + Resend patterns following the decisions above.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope.

</deferred>

---

*Phase: 04-forms-and-email*
*Context gathered: 2026-03-26*
