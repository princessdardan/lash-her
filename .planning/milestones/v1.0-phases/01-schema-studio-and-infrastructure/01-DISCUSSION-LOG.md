# Phase 1: Schema, Studio, and Infrastructure - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-03-24
**Phase:** 01-schema-studio-and-infrastructure
**Areas discussed:** Studio editing experience, Schema file organization, Block array design, Content model fidelity

---

## Studio Editing Experience

### Sidebar organization

| Option | Description | Selected |
|--------|-------------|----------|
| Grouped under "Pages" | 3-section sidebar: Pages (7 singletons), Content (training programs), Submissions (form data) | ✓ |
| Flat list with icons | All documents at top level with distinct icons | |
| Content vs Config split | Two groups: Content (editor-facing) and Configuration (structural/settings) | |

**User's choice:** Grouped under "Pages" — 3-section sidebar
**Notes:** Clean organization for daily editing. Pages / Content / Submissions grouping.

### Singleton behavior

| Option | Description | Selected |
|--------|-------------|----------|
| Direct open | Clicking singleton opens editor immediately, no list view | ✓ |
| List with single item | Standard list view showing one document | |

**User's choice:** Direct open
**Notes:** Uses documentListItem().child() pattern for cleaner editing flow.

### Document titles

| Option | Description | Selected |
|--------|-------------|----------|
| Friendly names | Human-readable: "Homepage", "Contact Page", etc. | ✓ |
| Match route names | Mirror URL routes: "Home", "Contact", "Gallery" | |

**User's choice:** Friendly names

### Block editing display

| Option | Description | Selected |
|--------|-------------|----------|
| Compact type list | Block type name + key field preview | ✓ |
| Expanded previews | More fields inline in array | |
| You decide | Let Claude pick | |

**User's choice:** Compact type list

---

## Schema File Organization

### Schema location

| Option | Description | Selected |
|--------|-------------|----------|
| frontend/src/sanity/schemas/ | Alongside Next.js code, follows next-sanity conventions | ✓ |
| sanity/ at project root | Top-level directory separate from frontend | |
| frontend/sanity/ (no src) | Inside frontend workspace but outside src/ | |

**User's choice:** frontend/src/sanity/schemas/
**Notes:** Full Sanity directory structure under frontend/src/sanity/ including lib/, studio/, and sanity.config.ts.

### Schema grouping

| Option | Description | Selected |
|--------|-------------|----------|
| By role: documents/ + objects/ | Two-way split matching Sanity's own distinction | ✓ |
| Three-way: documents/ + blocks/ + objects/ | Separate layout blocks from small reusable objects | |
| Flat directory | All files in one directory with naming prefixes | |

**User's choice:** By role: documents/ + objects/

---

## Block Array Design

### Block scope

| Option | Description | Selected |
|--------|-------------|----------|
| Per-page allowed blocks | Each page defines its own allowed block types | ✓ |
| Shared global set | All pages accept all 14 block types | |
| You decide | Let Claude pick based on Strapi config | |

**User's choice:** Per-page allowed blocks
**Notes:** Mirrors Strapi's dynamic zone configuration. Prevents editors from adding irrelevant blocks to pages.

### Field naming

| Option | Description | Selected |
|--------|-------------|----------|
| blocks | Same name as Strapi, minimizes Phase 2 changes | ✓ |
| pageBuilder | Common Sanity convention | |
| content | Generic name | |

**User's choice:** blocks

---

## Content Model Fidelity

### Training page structure

| Option | Description | Selected |
|--------|-------------|----------|
| Keep separate | trainingPage and trainingProgramsPage remain distinct | ✓ |
| Merge into one | Combine into single document | |

**User's choice:** Keep separate
**Notes:** Zero-risk migration path. Each maps to its own route.

### Unused section collection

| Option | Description | Selected |
|--------|-------------|----------|
| Skip it | Don't create schema for unused type | ✓ |
| Include it | Mirror for completeness | |

**User's choice:** Skip it
**Notes:** Already out-of-scope in REQUIREMENTS.md.

### Type naming cleanup

| Option | Description | Selected |
|--------|-------------|----------|
| Clean up names | Rename ambiguous types (e.g., layout 'contact-form' → 'contactFormLabels') | ✓ |
| Direct camelCase conversion | Mechanical conversion risking name collisions | |
| You decide | Let Claude resolve conflicts | |

**User's choice:** Clean up names
**Notes:** Avoids collision between layout block type and submission document type.

### Rich text fields

| Option | Description | Selected |
|--------|-------------|----------|
| Define as Portable Text now | Schema uses block type from day one | ✓ |
| Use plain text temporarily | Placeholder fields, replace in Phase 3 | |

**User's choice:** Define as Portable Text now
**Notes:** No schema migration needed later. Renderer comes in Phase 3 but schema is correct from Phase 1.

---

## Claude's Discretion

- Icon choices for sidebar document types
- Exact Portable Text block configuration (decorators, annotations, styles)
- Preview configuration for each schema type
- Internal structure of reusable sub-objects (field ordering, validation rules)

## Deferred Ideas

None — discussion stayed within phase scope.
