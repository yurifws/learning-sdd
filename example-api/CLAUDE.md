# CLAUDE.md — AI Onboarding

Read these files in this order before starting any task:

1. `constitution.md` — non-negotiable rules (architecture, naming, security, always/ask/never)
2. `spec.md` — architecture reference and full 6-layer code examples
3. `plan.md` — what is done, what is pending, architectural decisions
4. `task.md` — the ONE task you must implement right now

---

## After completing each task

1. Run `./mvnw test` — all tests must pass before marking done
2. Check every item in `task.md` § Done When — mark `[x]` only when verified
3. Update `plan.md` — mark the completed endpoint as ✅ Done
4. Update `spec.md` — only if an API contract, model, or layer pattern changed
5. Clear `task.md` § Active Task and § Files to Touch — ready for the next task

## If something is unclear

Stop. Do not guess. Ask before writing any code.
