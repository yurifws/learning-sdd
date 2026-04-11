# CLAUDE.md — AI Onboarding

Read these files in this order before starting any task:

1. `CONSTITUTION.md` — global rules, always apply, no exceptions
2. `CLARIFICATION_GATE.md` — ambiguity protocol, read before touching any spec
3. `LIVING_SPEC.md` — spec-first discipline, commit convention, Definition of Done
4. `specs/active/FEAT-XXX/requirements.md` — WHAT to build (EARS rules)
5. `specs/active/FEAT-XXX/design.md` — HOW to build it (architecture, DTOs, endpoints)
6. `specs/active/FEAT-XXX/lessons_learned.md` — known errors, check before debugging
7. `specs/active/FEAT-XXX/tasks.md` — ordered task list, start at the first unchecked task

---

## Rules during implementation

- Implement **one task at a time** — after each task, stop and wait for approval
- Never combine tasks or skip ahead without asking first
- If a task is ambiguous, flag it as `[NEEDS_CLARIFICATION]` and stop — never guess
- Log every unexpected error in `lessons_learned.md` before asking for help

---

## After each task is approved

1. Check off the task in `tasks.md` — `[ ]` → `[x]`
2. Run `./mvnw test` — all tests must pass
3. Update `lessons_learned.md` if any error was encountered and resolved
4. Wait for the next instruction — do not start the next task automatically

---

## When the feature is complete

> Full checklist → `LIVING_SPEC.md`

- [ ] `design.md` reflects the final behavior — no drift from implementation
- [ ] `lessons_learned.md` updated with any decisions made during implementation
- [ ] All tasks in `tasks.md` checked off
- [ ] All acceptance criteria in `requirements.md` Section 6 pass
- [ ] No unresolved `[NEEDS_CLARIFICATION]` items
- [ ] `./mvnw test` passes — zero failures, coverage ≥ 80%
- [ ] Zero unresolved TODOs or FIXMEs in changed files
- [ ] Ready for IMPL PR
