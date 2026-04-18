# AGENTS.md — Task Management API

> This is a filled-in example. See the blank template at `../AGENTS.md`.

Read this file first. Then read the three files below in order before doing anything else.

---

## Purpose

This folder is the persistent brain of the Task Management API project. Any AI agent — in any session — reads these three files and is fully onboarded. No re-explaining. No guessing.

---

## Reading Order

Read in this exact order before touching any code, spec, or task:

1. [PROJECT_PLAN.md](PROJECT_PLAN.md) — what we are building (Task API) and what we are NOT (no frontend, no attachments, no real-time)
2. [PROJECT_ROADMAP.md](PROJECT_ROADMAP.md) — we are in Phase 1 (Core Task API); Phase 2 is Assignment
3. [PROJECT_TECHSTACK.md](PROJECT_TECHSTACK.md) — Java 21, Spring Boot 3.3, Hexagonal Architecture, MapStruct, no Lombok
4. [CONSTITUTION.md](CONSTITUTION.md) — non-negotiable architecture, naming, and security rules; always/ask/never for Claude
5. [CLARIFICATION_GATE.md](CLARIFICATION_GATE.md) — all ambiguities resolved before spec; 5 buckets (auth, shape, errors, edge cases, ownership)
6. [requirements.md](requirements.md) — EARS requirements for Phase 1 (what to build)
7. [design.md](design.md) — architecture decisions, data model, endpoints, sequence diagrams (how to build it)
8. [LIVING_SPEC.md](LIVING_SPEC.md) — spec-first discipline; 4-step rule; commit convention; Definition of Done
9. [TEST_FIRST_GATE.md](TEST_FIRST_GATE.md) — all 9 tests confirmed failing before implementation begins
10. [tasks.md](tasks.md) — ordered task list; start at the first unchecked task

---

## Rules

- **Never guess** what the project does — it is in `PROJECT_PLAN.md`.
- **Never assume** priority — current phase and backlog order are in `PROJECT_ROADMAP.md`.
- **Never substitute** libraries or patterns — the stack is locked in `PROJECT_TECHSTACK.md`.
- **Never put business logic** in a controller or persistence service — use cases only.
- **Never use Lombok** — Java records for DTOs, plain classes for domain objects.
- **Never start implementation without a passing test-first gate** — see `TEST_FIRST_GATE.md`.
- If something contradicts these files, **stop and flag it** with `[NEEDS_CLARIFICATION]` before writing any code.

---

## When to Update These Files

| File | Update when... |
|---|---|
| `PROJECT_PLAN.md` | Scope changes, new "not building" decisions, success criteria change |
| `PROJECT_ROADMAP.md` | A phase completes, priorities shift, something gets frozen or unfrozen |
| `PROJECT_TECHSTACK.md` | A library is added, upgraded, or banned |
| `CONSTITUTION.md` | A new non-negotiable rule is agreed on by the team |
| `CLARIFICATION_GATE.md` | A new ambiguity is resolved — add the Q&A before implementing |
| `requirements.md` | A requirement is added, changed, or removed |
| `design.md` | A new endpoint, DTO field, exception type, or architectural decision is added |
| `tasks.md` | A new task is added, or an existing task's scope changes |

> Rule: update the file **before** implementing the change. These files are always the source of truth.
> Spec order: `requirements.md` → `design.md` → `tasks.md` → code. Never skip steps.