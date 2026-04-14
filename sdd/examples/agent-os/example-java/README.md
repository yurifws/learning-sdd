# example-java — Task Management API

> A complete walkthrough of the multi-agent contract-first workflow applied to a real feature.
> **Feature:** Phase 1 — Core Task API (5 endpoints, JWT auth, hexagonal architecture)
> **Stack:** Java 21 · Spring Boot 3.3 · PostgreSQL · Flyway · MapStruct

---

## What This Example Shows

This is a filled-in example of the agent-os pattern. It demonstrates:

- **Persistent project brain** — `PROJECT_PLAN.md`, `PROJECT_TECHSTACK.md`, `PROJECT_ROADMAP.md` keep every agent oriented across sessions
- **Orchestrator + specialist agents** — one orchestrator reads the full plan; each specialist sees only its layer
- **Contract-first handoff** — each specialist produces a locked contract before the next specialist starts
- **Human review gates** — four hard stops where a human approves before work continues

The folder is a working snapshot: all spec files are filled in, contracts are written, tasks are ready to feed to an AI one at a time.

---

## Reading Order

```
1. PROJECT_PLAN.md          ← scope, mission, what we are NOT building
2. PROJECT_TECHSTACK.md     ← locked stack — every agent reads this first
3. requirements.md          ← EARS rules — what the system must do
4. design.md                ← architecture, DTOs, endpoints, transitions
5. CLARIFICATION_GATE.md    ← all ambiguity resolved before any code
6. ORCHESTRATION.md         ← how to split this feature across specialist agents
7. db-contract.md           ← contract produced by the Persistence Specialist (read before Use Case)
8. api-contract.md          ← contract produced by the Use Case Specialist (read before Controller)
9. tasks.md                 ← atomic task list for sequential or orchestrated execution
10. TEST_FIRST_GATE.md      ← confirmed failing tests before implementation starts
```

---

## The Contract Chain

This feature has four specialist layers. Each one produces a contract before the next one starts:

```
┌──────────────────────────────────────────────────────────────────┐
│  ORCHESTRATOR                                                    │
│  Reads: PROJECT_PLAN, PROJECT_TECHSTACK, requirements, design,   │
│         CLARIFICATION_GATE, tasks                                │
│  Delegates to specialists in order (sequential — each layer      │
│  depends on the one below it)                                    │
└───────────────────────────┬──────────────────────────────────────┘
                            │
          ┌─────────────────▼──────────────────────┐
          │  DOMAIN SPECIALIST                      │
          │  Reads: PROJECT_TECHSTACK, design §6    │
          │  Produces: TaskDomain.java, TaskStatus  │
          │  Human Gate A: domain logic reviewed ✓  │
          └─────────────────┬──────────────────────┘
                            │
          ┌─────────────────▼──────────────────────┐
          │  PERSISTENCE SPECIALIST                 │
          │  Reads: PROJECT_TECHSTACK, design §3,   │
          │         TaskDomain (from Gate A)        │
          │  Produces: db-contract.md (LOCKED)      │
          │  Human Gate B: DB schema + port ✓       │
          └─────────────────┬──────────────────────┘
                            │ reads db-contract.md
          ┌─────────────────▼──────────────────────┐
          │  USE CASE SPECIALIST                    │
          │  Reads: PROJECT_TECHSTACK, requirements,│
          │         db-contract.md                  │
          │  Produces: api-contract.md (LOCKED)     │
          │  Human Gate C: application logic ✓      │
          └─────────────────┬──────────────────────┘
                            │ reads api-contract.md
          ┌─────────────────▼──────────────────────┐
          │  CONTROLLER SPECIALIST                  │
          │  Reads: PROJECT_TECHSTACK, design §4-5, │
          │         api-contract.md                 │
          │  Produces: implemented REST endpoints   │
          │  Human Gate D: full suite passes ✓      │
          └─────────────────────────────────────────┘
```

---

## The Four Human Gates

| Gate                | When                                 | What you review                                     | How to pass                                             |
| ------------------- | ------------------------------------ | --------------------------------------------------- | ------------------------------------------------------- |
| **A — Domain**      | After Domain Specialist finishes     | `TaskDomain.java` — status transitions, field types | `./mvnw test -Dtest=TaskDomainTest` green               |
| **B — Persistence** | After `db-contract.md` is written    | Table schema, port-out method signatures            | Review `db-contract.md` — does it match `design.md §3`? |
| **C — Use Case**    | After `api-contract.md` is written   | Port-in signatures, request/response shapes         | `./mvnw test -Dtest=TaskUseCaseTest` green              |
| **D — Full suite**  | After Controller Specialist finishes | All 5 endpoints behave per requirements             | `./mvnw verify` — zero failures                         |

You are not reviewing every line of code at each gate. You are confirming the **contract** matches the spec before the next agent consumes it. Catching a wrong method signature at Gate B costs five minutes. Catching it at Gate D costs an hour.

---

## How to Use This Example

**To understand the pattern:** Read the files in the order above. The contracts (`db-contract.md`, `api-contract.md`) are the key artifacts — notice what information each specialist needs, and how little it needs from layers above or below.

**To use it as a template:** Copy this folder, replace the Task Management content with your own feature, fill in the `[FILL]` placeholders in `PROJECT_PLAN.md` and `PROJECT_TECHSTACK.md`, then follow the reading order.

**To run it with an AI:** Use the prompt templates at the bottom of `tasks.md`. Feed one task at a time. Wait for the verify command to pass before approving the next task.

---

## Files in This Folder

| File                    | What it is                                                         |
| ----------------------- | ------------------------------------------------------------------ |
| `PROJECT_PLAN.md`       | Mission, scope, success criteria                                   |
| `PROJECT_ROADMAP.md`    | Phase milestones                                                   |
| `PROJECT_TECHSTACK.md`  | Locked stack + banned patterns                                     |
| `AGENTS.md`             | AI onboarding — reading order + post-task checklist                |
| `CONSTITUTION.md`       | Non-negotiable architectural rules                                 |
| `requirements.md`       | EARS requirements for Phase 1                                      |
| `design.md`             | Architecture, data model, DTOs, endpoints, transitions             |
| `CLARIFICATION_GATE.md` | Resolved ambiguities (approved before spec was written)            |
| `ORCHESTRATION.md`      | Orchestrator + specialist prompts for this feature                 |
| `db-contract.md`        | **Contract** produced by Persistence Specialist — locked at Gate B |
| `api-contract.md`       | **Contract** produced by Use Case Specialist — locked at Gate C    |
| `tasks.md`              | Atomic task list with verify commands                              |
| `TEST_FIRST_GATE.md`    | Evidence that all tests fail before implementation                 |
