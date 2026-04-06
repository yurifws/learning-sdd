## What does this PR do?

<!-- Link the spec or issue this implements -->
**Feature ID:** FEAT-XXX
**Issue:** #

---

## Type of change

- [ ] 📋 SPEC — requirements.md / design.md only (no code)
- [ ] 💻 IMPL — implementation of approved spec
- [ ] 🐛 Fix — bug fix
- [ ] 🔧 Chore — refactor, tests, config

---

## Spec checklist (SPEC PRs)

- [ ] requirements.md uses EARS notation for all rules
- [ ] All unwanted behavior cases covered in Section 2.6
- [ ] No `[NEEDS_CLARIFICATION]` items remaining
- [ ] design.md matches requirements.md
- [ ] tasks.md reviewed and ordered

---

## Implementation checklist (IMPL PRs)

- [ ] All tasks in tasks.md checked off
- [ ] `./mvnw test` passes — zero failures, coverage ≥ 80%
- [ ] Code follows CONSTITUTION.md rules
- [ ] No business logic in Controller layer
- [ ] No entity objects returned directly from Controller (use DTOs)
- [ ] All responses use `ApiResponse<T>` envelope
- [ ] All acceptance criteria from requirements.md Section 6 pass
- [ ] Unit tests cover all Service methods (happy path + error cases)
- [ ] Integration tests cover all Controller endpoints
- [ ] design.md reflects final behavior — no drift from implementation
- [ ] lessons_learned.md updated if any errors occurred
- [ ] Commits follow convention: spec → plan → implementation
- [ ] No sensitive data logged
- [ ] No stack traces exposed in API responses
