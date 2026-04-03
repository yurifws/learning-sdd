# learning-sdd

A learning repository for **SDD (Software-Driven Development)** — a methodology to control AI agents when building software. The goal is to give AI exactly what it needs, nothing more, and keep every change traceable back to a requirement.

---

## What is SDD?

SDD is a workflow where:
1. You define the **rules** before the AI touches any code (Constitution)
2. You define **what to build** with explicit contracts (SDD / Spec)
3. You break work into **atomic tasks** — one objective, 1–2 files each
4. You feed the AI **one task at a time** — guided execution, no drift
5. Every change is **verified** before it is accepted

---

## Repository Structure

```
learning-sdd/
├── PROJECT_CONSTITUTION.md        # Generic template: non-negotiable rules for any project
├── PROJECT_CONSTITUTION_JAVA.md   # Same template, specialized for Java/Spring Boot
├── SDD_REST_API_TEMPLATE.md       # Generic template: software design document for REST APIs
├── TASKS.md                       # Generic template: atomic task list with verify commands
├── GUIDED_EXECUTION.md            # Rules for how to feed tasks to the AI safely
└── example-api/
    ├── constitution.md            # Filled-in constitution for the example Spring Boot API
    ├── spec.md                    # Filled-in spec: hexagonal package structure, endpoints, code examples
    ├── plan.md                    # Current implementation status and architectural decisions
    └── task.md                    # Active task: what the AI must implement right now
```

---

## The SDD Flow

```
Constitution  →  defines non-negotiable rules (architecture, naming, security)
     ↓
SDD / Spec    →  defines what to build (endpoints, data models, success criteria)
     ↓
Tasks         →  breaks the spec into atomic steps (1 objective, 1-2 files, verify command)
     ↓
Guided AI     →  receives one task at a time, only the files listed
     ↓
Verify        →  acceptance criteria must pass before the task is marked done ✅
```

---

## Example API

The `example-api/` folder contains a concrete **Spring Boot REST API** built with Hexagonal Architecture (Ports & Adapters). It serves as the reference implementation for the SDD methodology.

### Tech Stack
- Java 17 + Spring Boot
- Hexagonal Architecture (Ports & Adapters)
- Spring Data JPA + H2 (dev) / PostgreSQL (prod)
- MapStruct for mapping
- Spring Security with `@PreAuthorize`
- OpenAPI 3 (Swagger)

### Endpoints
| Method | Endpoint | Status |
|--------|----------|--------|
| GET | `/clients/{id}/resumo` | ✅ Done |
| GET | `/clients/{id}/enderecos` | ⏳ Pending |
| GET | `/clients/{id}/contatos` | ⏳ Pending |
| PUT | `/clients/{id}/plano` | ⏳ Pending |

### Hexagonal Layers (per endpoint)
Every endpoint is implemented across 6 layers:
1. **Controller** — handles HTTP, no business logic
2. **OpenApi Interface** — Swagger contract + `@PreAuthorize`
3. **Port In** — input port interface
4. **Use Case** — business logic
5. **Port Out** — output port interface
6. **Persistence Service** — JPA repository calls

---

## Key Principles

- **Incremental disclosure** — give the AI only the files needed for the current task
- **One task = one PR** — if reviewing hurts your brain, the task was too big
- **Drift detection** — any file touched outside the task list is a hard stop
- **Audit trail** — every change traces back: Requirement → Task → PR → Acceptance Criteria
