# Project Retrospective

*A living document updated after each milestone. Lessons feed forward into future planning.*

## Milestone: v1.0 — Strapi to Sanity Migration

**Shipped:** 2026-03-30
**Phases:** 6 | **Plans:** 17 | **Tasks:** 33

### What Was Built
- Complete CMS migration: 33 Sanity schemas, 10 GROQ loaders, embedded Studio at /studio
- Portable Text renderer with brand-styled typography replacing Strapi Blocks rich text
- Form pipeline: Server Actions → Sanity documents + Resend emails (4 HTML templates)
- On-demand ISR via Sanity webhook with HMAC signature verification
- Automated migration script (1,259 lines): content, images, and rich text transformation
- Full Strapi removal: backend directory, packages, env vars, and all code references

### What Worked
- **Dependency-ordered phases**: Schema → Data → Rich Text/Forms (parallel) → Cache → Migration. Each phase built cleanly on the previous foundation with minimal rework
- **Big bang cutover strategy**: Small content volume made a single-swap migration viable — no dual-CMS complexity
- **Gap closure plans**: Phase 2 added two unplanned plans (02-05, 02-06) to fix TypeScript compilation issues discovered after the main plans. Catching these before moving on prevented cascading debt
- **Stub-then-implement pattern**: Phase 2 stubbed Portable Text and form submissions, Phase 3/4 replaced stubs — clean separation of concerns across phases
- **Automated migration script**: Handled all 10 content types, image deduplication, and rich text conversion in one repeatable run

### What Was Inefficient
- **REQUIREMENTS.md tracking went stale**: 10 requirements stayed unchecked despite being completed — traceability table wasn't updated during phase transitions. The checkboxes and traceability Pending/Complete status drifted from reality
- **Phase 2 scope underestimated**: Originally planned for 4 plans but needed 6 (two gap closures for TypeScript errors). Better type-checking during planning could have caught these
- **npm uninstall blocked by token-gated URLs**: motion-plus token-gated registry URLs caused npm uninstall to fail for unrelated packages — had to edit package.json directly
- **No milestone audit**: Skipped `/gsd:audit-milestone` — would have caught the stale requirements earlier

### Patterns Established
- **Singleton pattern**: documentId matches type name (e.g., homePage/homePage) — simple, predictable
- **camelCase _type naming**: Established in Phase 1, used consistently across all 33 schema types
- **Fetch-level cache tags**: Applied directly to GROQ fetch calls instead of unstable_cache wrappers — simpler model
- **assertValue for required env vars**: App crashes at startup if missing, preventing silent security degradation
- **Promise.allSettled for email**: Non-blocking email sending that never rejects — form submission succeeds even if email fails

### Key Lessons
1. **Update traceability during transitions, not at milestone end** — stale checkboxes create false signals about completion status
2. **TypeScript strict compilation is its own verification gate** — run `tsc --noEmit` after each data-layer change, not just at plan end
3. **Gap closure plans are normal, not failure** — budget 1-2 unplanned plans per complex phase
4. **Token-gated npm registries break standard tooling** — when packages use custom registries, edit package.json directly instead of `npm uninstall`

### Cost Observations
- Model mix: primarily opus for execution, sonnet for research/planning agents
- Sessions: ~12 sessions over 6 days
- Notable: Parallel phase execution (3+4 alongside 2) saved wall-clock time

---

## Cross-Milestone Trends

### Process Evolution

| Milestone | Phases | Plans | Key Change |
|-----------|--------|-------|------------|
| v1.0 | 6 | 17 | Initial migration — established schema-first, dependency-ordered workflow |

### Top Lessons (Verified Across Milestones)

1. Schema-first development pays off — every downstream phase benefits from a stable foundation
2. Stub patterns enable clean phase separation without blocking parallel work
