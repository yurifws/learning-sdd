# AGENTS.md — AI Onboarding

Read these files in this order before starting any task:

1. `CONSTITUTION.md` — non-negotiable rules (architecture, naming, security, always/ask/never)
2. `CLARIFICATION_GATE.md` — ambiguity protocol; read before touching any spec or task
3. `requirements.md` — EARS requirements for all endpoints (what to build)
4. `spec.md` — architecture reference and full 6-layer code examples (how to build it)
5. `plan.md` — what is done, what is pending, architectural decisions
6. `tasks.md` — ordered task list and active task detail (acceptance criteria, files to touch)

---

## After completing each task

1. Run `./mvnw test` — all tests must pass before marking done
2. Check every item in `tasks.md` § Done When — mark `[x]` only when verified
3. Update `plan.md` — mark the completed endpoint as ✅ Done
4. Update `tasks.md` — mark the task `[x]` and clear the Active Task section
5. Update `spec.md` — only if an API contract, model, or layer pattern changed

## If something is unclear

Stop. Flag it as `[NEEDS_CLARIFICATION]`. Do not guess. Do not write any code until it is resolved.
Re-read `CLARIFICATION_GATE.md` and go through the 5-bucket protocol before resuming.
