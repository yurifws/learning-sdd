# CLAUDE.md — AI Onboarding

Read these files in this order before starting any task:

1. `CONSTITUTION.md` — global rules, always apply, no exceptions
2. `specs/active/FEAT-XXX/requirements.md` — WHAT to build (EARS rules)
3. `specs/active/FEAT-XXX/design.md` — HOW to build it (architecture, DTOs, endpoints)
4. `specs/active/FEAT-XXX/lessons_learned.md` — known errors, check before debugging
5. `specs/active/FEAT-XXX/tasks.md` — ordered task list, start at the first unchecked task

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

- All tasks in `tasks.md` checked off
- All acceptance criteria in `requirements.md` Section 6 verified
- `lessons_learned.md` updated with any decisions made during implementation
- Ready for IMPL PR
