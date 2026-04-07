# AGENTS.md — Task Management API

> This is a filled-in example. See the blank template at `agent-os/AGENTS.md`.

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

---

## Rules

- **Never guess** what the project does — it is in `PROJECT_PLAN.md`.
- **Never assume** priority — current phase and backlog order are in `PROJECT_ROADMAP.md`.
- **Never substitute** libraries or patterns — the stack is locked in `PROJECT_TECHSTACK.md`.
- **Never put business logic** in a controller or persistence service — use cases only.
- **Never use Lombok** — Java records for DTOs, plain classes for domain objects.
- If something contradicts these files, **stop and flag it** with `[NEEDS_CLARIFICATION]` before writing any code.

---

## When to Update These Files

| File | Update when... |
|---|---|
| `PROJECT_PLAN.md` | Scope changes, new "not building" decisions, success criteria change |
| `PROJECT_ROADMAP.md` | A phase completes, priorities shift, something gets frozen or unfrozen |
| `PROJECT_TECHSTACK.md` | A library is added, upgraded, or banned |

> Rule: update the file **before** implementing the change. These files are always the source of truth.