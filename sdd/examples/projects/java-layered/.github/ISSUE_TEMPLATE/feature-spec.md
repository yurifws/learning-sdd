---
name: New Feature Spec
about: Start a new feature using the SDD workflow
title: "[FEAT-XXX] "
labels: spec
---

## Feature Summary

<!-- One sentence: what does this feature do and why? -->

**Feature ID:** FEAT-XXX
**Priority:** High | Medium | Low
**Team / Stakeholder:** [Team Name]

---

## Initial Requirements (rough)

<!-- Don't worry about EARS notation yet — just describe the behavior in plain language -->

- When...
- The system should...
- It should never...

---

## Acceptance Criteria (draft)

<!-- These will be formalized in requirements.md Section 6 -->

- [ ] ...
- [ ] ...

---

## Out of Scope

<!-- What this feature explicitly does NOT include -->

- ...

---

## Open Questions

<!-- Blockers that must be resolved before writing the spec -->

- [ ] ...

---

## Next Step

1. Create branch `feat/FEAT-XXX-short-name`
2. Apply `CLARIFICATION_GATE.md` — flag and resolve all `[NEEDS_CLARIFICATION]` items before writing any spec
3. Create `specs/active/FEAT-XXX/`:
   - `requirements.md` — EARS notation, acceptance criteria
   - `design.md` — architecture, DTOs, endpoints, sequences
   - `tasks.md` — ordered task list with verify commands
   - `lessons_learned.md` — start empty, update during implementation
4. Open a **SPEC PR** for team review before writing any code
