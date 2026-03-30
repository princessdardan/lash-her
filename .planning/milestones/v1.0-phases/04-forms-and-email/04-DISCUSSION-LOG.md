# Phase 4: Forms and Email - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-03-26
**Phase:** 04-forms-and-email
**Areas discussed:** Server Action layout, Email template migration, Submission error handling, Write token scoping

---

## Server Action Layout

### Q1: How should the Server Action files be organized?

| Option | Description | Selected |
|--------|-------------|----------|
| Single shared action file | One file at frontend/src/app/actions/form.ts exporting submitGeneralInquiry() and submitTrainingContact(). Shared helpers as private functions. | ✓ |
| Per-form action files | Separate files: actions/general-inquiry.ts and actions/training-contact.ts. Each self-contained. | |
| You decide | Claude picks the best approach during planning. | |

**User's choice:** Single shared action file
**Notes:** Recommended option — keeps it simple for 2 forms with shared helpers.

### Q2: Should the action return typed result objects, or use Next.js form state conventions?

| Option | Description | Selected |
|--------|-------------|----------|
| Typed result object | Return { success: boolean; error?: string }. Form components call the action directly. | ✓ |
| useActionState integration | Use React 19's useActionState hook for form state management. | |
| You decide | Claude picks the best approach during planning. | |

**User's choice:** Typed result object
**Notes:** Simple, explicit, matches existing patterns.

### Q3: Should Server Actions also validate server-side?

| Option | Description | Selected |
|--------|-------------|----------|
| Server-side validation too | Re-validate in the Server Action using the same validation rules. Defense in depth. | ✓ |
| Trust client validation | Only validate on the client. Server Action assumes data is clean. | |
| You decide | Claude picks the best approach during planning. | |

**User's choice:** Server-side validation too
**Notes:** Defense in depth — client validation is bypassable.

---

## Email Template Migration

### Q1: How should the 4 HTML email templates move from the Strapi backend to the frontend?

| Option | Description | Selected |
|--------|-------------|----------|
| Exact port | Copy the 4 templates from backend/src/services/email.ts into frontend/src/lib/email.ts. Same HTML, same styling. | ✓ |
| Refactor to React Email | Rewrite templates using @react-email/components for type-safe emails. | |
| You decide | Claude picks the best approach during planning. | |

**User's choice:** Exact port
**Notes:** Zero regression in email appearance.

### Q2: Should the email module be a class or plain exported functions?

| Option | Description | Selected |
|--------|-------------|----------|
| Plain functions | Export sendFormEmails(), sendAdminNotification(), sendUserConfirmation() as standalone functions. | ✓ |
| Keep class pattern | Port the EmailService class as-is. | |
| You decide | Claude picks the best approach during planning. | |

**User's choice:** Plain functions
**Notes:** Simpler for stateless Server Action calls.

---

## Submission Error Handling

### Q1: If the Sanity document write succeeds but emails fail, what should the user see?

| Option | Description | Selected |
|--------|-------------|----------|
| Success | Show success message. Email failure logged server-side but invisible to user. | ✓ |
| Success with caveat | Show success but add a note about possible email delay. | |
| You decide | Claude picks the best approach during planning. | |

**User's choice:** Success
**Notes:** Matches current Strapi non-blocking pattern.

### Q2: If the Sanity document write itself fails, what should happen?

| Option | Description | Selected |
|--------|-------------|----------|
| Show error, no emails | Return generic error. Don't send emails since nothing was saved. Log server-side. | ✓ |
| Show error with details | Return specific error based on failure type. | |
| You decide | Claude picks the best approach during planning. | |

**User's choice:** Show error, no emails
**Notes:** No internal details exposed to the user.

---

## Write Token Scoping

### Q1: How should the Sanity write token be scoped for form submissions (FORM-04)?

| Option | Description | Selected |
|--------|-------------|----------|
| Separate form token | Dedicated SANITY_FORM_TOKEN with create-only permissions on contactForm and generalInquiry. | ✓ |
| Reuse existing write token | Use SANITY_WRITE_TOKEN for form submissions too. | |
| You decide | Claude picks the best approach during planning. | |

**User's choice:** Separate form token
**Notes:** Principle of least privilege.

### Q2: Should the form write client be a separate module from the general write client?

| Option | Description | Selected |
|--------|-------------|----------|
| Separate form-client.ts | Create frontend/src/sanity/lib/form-client.ts using SANITY_FORM_TOKEN. | ✓ |
| Parameterize existing write client | Add optional token parameter to existing write client factory. | |
| You decide | Claude picks the best approach during planning. | |

**User's choice:** Separate form-client.ts
**Notes:** Keeps form submission code isolated from general write client.

---

## Claude's Discretion

- Internal Server Action implementation details
- Exact server-side validation implementation
- Resend client initialization pattern
- Environment variable naming for Resend config
- Whether to add rate limiting or spam protection

## Deferred Ideas

None — discussion stayed within phase scope.
