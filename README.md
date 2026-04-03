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
│
│   ── Generic templates (stack-agnostic, copy & fill for any project)
├── PROJECT_CONSTITUTION.md        # Rulebook template: architecture, naming, security gates, do-not list
├── SDD_REST_API_TEMPLATE.md       # Design document template: endpoints, data models, EARS criteria
├── TASKS.md                       # Task list template: one objective per task, verify command, parallelism hints
├── GUIDED_EXECUTION.md            # AI execution rules: incremental disclosure, drift detection, Definition of Done
├── AGENTS.md                      # Template for writing an AGENTS.md for any project
│
│   ── Stack-specific templates (ready-to-use for Java/Spring Boot)
├── BACKEND_JAVA/
│   ├── PROJECT_CONSTITUTION_JAVA.md   # Constitution with Java/Spring Boot rules, testing, migrations
│   └── AGENTS_BACKEND_JAVA.md         # AI onboarding for hexagonal Spring Boot: layers, naming, conventions
│
│   ── Concrete example (filled-in SDD for a real Spring Boot API)
└── example-api/
    ├── CLAUDE.md                  # AI entrypoint: read these files in order before starting
    ├── constitution.md            # Filled-in rules for this specific project
    ├── spec.md                    # Full spec: hexagonal layers, endpoints, reference code for all 6 layers
    ├── plan.md                    # Implementation progress and architectural decisions
    └── task.md                    # Active task: what to implement right now
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
| GET | `/clients/{id}/summary` | ✅ Done |
| GET | `/clients/{id}/addresses` | ⏳ Pending |
| GET | `/clients/{id}/contacts` | ⏳ Pending |
| PUT | `/clients/{id}/plan` | ⏳ Pending |

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
