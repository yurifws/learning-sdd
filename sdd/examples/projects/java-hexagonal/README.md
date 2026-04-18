# Client Management API — SDD Hexagonal Example

> A full-feature SDD example applied to Java + Spring Boot using **Hexagonal Architecture (Ports & Adapters)**.
> Read this when you need to see SDD applied to a domain with multiple input adapters (REST + gRPC) and strict layer isolation.

---

## What this is

A client-management API specified end-to-end with the SDD templates. One feature is fully specified: **FEAT-001 — Client Summary + gRPC adapter**. Demonstrates enterprise patterns: Oracle-style table naming, `@PreAuthorize` security, MapStruct mapping, OpenAPI docs, and domain POJOs with zero framework annotations.

Related examples in this folder:

- [`../reference-project/`](../reference-project/) — the same workflow, smaller scope, Python + FastAPI
- [`../java-layered/`](../java-layered/) — Java + Spring Boot with Layered architecture (Controller → Service → Repository)

---

## Why hexagonal here

Choose hexagonal when:
- The domain is complex enough that you'll want it isolated from HTTP, DB, and messaging concerns
- You expect **multiple input adapters** (REST + gRPC + events) over the same use cases
- You want to test the domain without any Spring context loaded

If this isn't you, start with [`../java-layered/`](../java-layered/) — it's faster to read and faster to ship.

---

## Folder layout

```
java-hexagonal/
├── CONSTITUTION.md              ← rules specific to this project (layering, security, DB)
├── AGENTS.md                    ← AI onboarding: commands, package map, boundaries
├── CLARIFICATION_GATE.md        ← ambiguity log — resolved before the spec was locked
└── specs/
    └── active/
        └── FEAT-001/
            ├── requirements.md      ← EARS behavior rules for all endpoints
            ├── design.md            ← hexagonal architecture, ports, adapters, data model
            ├── TEST_FIRST_GATE.md   ← failing tests recorded before implementation
            └── tasks.md             ← ordered task list
```

> **Note:** this example does not include a separate `scenarios.md`. The hexagonal project embeds Given/When/Then coverage inside `requirements.md` and the failing-test evidence inside `TEST_FIRST_GATE.md`. For a project that uses a standalone `scenarios.md`, see [`../reference-project/`](../reference-project/).

---

## Reading order

1. `CONSTITUTION.md` — non-negotiable rules (layering, tables, security)
2. `AGENTS.md` — what the AI reads first in every session
3. `CLARIFICATION_GATE.md` — questions asked and resolved before the spec
4. `specs/active/FEAT-001/requirements.md` — EARS rules for the feature
5. `specs/active/FEAT-001/design.md` — ports, adapters, persistence mapping
6. `specs/active/FEAT-001/TEST_FIRST_GATE.md` — failing tests recorded
7. `specs/active/FEAT-001/tasks.md` — ordered implementation tasks

---

## Key design patterns to study

- **Ports before adapters** — `design.md` defines `ClientPersistencePortOut` as an interface before either the use case or the JPA adapter exists. See [`../../patterns/agent-os/CONTRACT_FLOW.md`](../../patterns/agent-os/CONTRACT_FLOW.md) for the full contract-first pattern.
- **Domain isolation** — classes in `domain/` have zero framework annotations (no `@Entity`, no `@Component`). Adapters map to/from domain types.
- **Multiple input adapters** — `adapters/input/controller/` (REST) and `adapters/input/grpc/` (gRPC) call the same use case. The use case doesn't know or care which one called it.
