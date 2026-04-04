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

Create branch `feat/FEAT-XXX-short-name` and write the spec files:

- `specs/active/FEAT-XXX/requirements.md` — EARS notation, acceptance criteria
- `specs/active/FEAT-XXX/design.md` — architecture, DTOs, endpoints, sequences
- `specs/active/FEAT-XXX/tasks.md` — ordered task list with verify commands
- `specs/active/FEAT-XXX/lessons_learned.md` — start empty, update during implementation

Then open a **SPEC PR** for team review before writing any code.
