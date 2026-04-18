# Notification Service — SDD Reference Project

> The cleanest, smallest example of every SDD template filled in for a real feature.
> Start here if you want to see what a full spec-to-tasks workflow looks like before you apply it yourself.

---

## What this is

A notification service (email + SMS) built in Python + FastAPI, specified end-to-end using the SDD templates from [`../../..`](../../..). One feature is fully specified: **FEAT-001 — Send Notification**.

This project uses the generic, stack-agnostic templates unchanged. If you're comparing stacks, see also:

- [`../java-layered/`](../java-layered/) — same layered architecture, in Java + Spring Boot
- [`../java-hexagonal/`](../java-hexagonal/) — hexagonal architecture (ports & adapters), in Java + Spring Boot

---

## Folder layout

```
reference-project/
├── CONSTITUTION.md              ← non-negotiable rules for this project
├── AGENTS.md                    ← AI onboarding: commands, project map, boundaries
├── CLARIFICATION_GATE.md        ← ambiguity resolution log — all items resolved
└── specs/
    └── active/
        └── FEAT-001/
            ├── requirements.md      ← EARS behavior rules
            ├── design.md            ← architecture, DTOs, data model
            ├── scenarios.md         ← Given/When/Then derived from requirements
            ├── TEST_FIRST_GATE.md   ← tests confirmed failing before code
            └── tasks.md             ← ordered task list with verify commands
```

---

## Reading order

1. `CONSTITUTION.md` — the rules the AI must never break
2. `AGENTS.md` — what the AI reads at the start of every session
3. `CLARIFICATION_GATE.md` — the questions asked before the spec was locked
4. `specs/active/FEAT-001/requirements.md` — the locked EARS requirements
5. `specs/active/FEAT-001/design.md` — architecture and interface choices
6. `specs/active/FEAT-001/scenarios.md` — Given/When/Then for happy, edge, error
7. `specs/active/FEAT-001/TEST_FIRST_GATE.md` — failing tests before implementation
8. `specs/active/FEAT-001/tasks.md` — the ordered, atomic task list

---

## How to use this example

- **To learn the workflow:** read top-to-bottom in the order above. Every file cross-references the next.
- **To start your own project:** copy this folder, rename `FEAT-001` to your feature ID, and clear the content — leave the structure. Then follow [`../../../../PROJECT_BOOTSTRAP.md`](../../../../PROJECT_BOOTSTRAP.md).
- **To compare architectures:** open `requirements.md` here and in `../java-hexagonal/specs/active/FEAT-001/requirements.md`. The EARS clauses are the same shape; only the design and tasks change per architecture.
