# Project Examples

> Fully worked SDD projects — every template filled in for a real feature, on a real stack.  
> Use these when you want to see what a complete spec-to-tasks workflow looks like in practice.

---

## Examples

| Project | Stack | Architecture | Feature shown |
|---------|-------|--------------|---------------|
| [`reference-project/`](reference-project/) | Python · FastAPI · PostgreSQL | Layered | Send Notification (email/SMS) — the cleanest starting point |
| [`java-layered/`](java-layered/) | Java 17 · Spring Boot · PostgreSQL | Layered | Task CRUD with JWT auth |
| [`java-hexagonal/`](java-hexagonal/) | Java 17 · Spring Boot · H2/PostgreSQL | Hexagonal (Ports & Adapters) | Client management with gRPC adapter |

---

## How to Use These Examples

Each project folder contains a complete set of SDD files. The reading order within any project is always:

1. `CONSTITUTION.md` — non-negotiable rules for this project
2. `AGENTS.md` — AI onboarding: commands, project map, conventions, boundaries
3. `CLARIFICATION_GATE.md` — all ambiguities resolved before specs were written
4. `specs/active/FEAT-XXX/requirements.md` — EARS requirements
5. `specs/active/FEAT-XXX/design.md` — architecture and data model
6. `specs/active/FEAT-XXX/scenarios.md` — Given/When/Then derived from requirements
7. `specs/active/FEAT-XXX/TEST_FIRST_GATE.md` — tests confirmed failing before implementation
8. `specs/active/FEAT-XXX/tasks.md` — ordered task list with verify commands

---

## Which Example to Start With

**New to SDD?** → Start with [`reference-project/`](reference-project/). It uses the generic templates exactly as they appear in `/sdd`, filled in for a non-Java, non-TypeScript stack. Every file shows what the placeholder text becomes in a real project.

**Building a Spring Boot CRUD API?** → [`java-layered/`](java-layered/) is the fastest path. Copy the structure, swap the domain.

**Complex domain with multiple adapters?** → [`java-hexagonal/`](java-hexagonal/) shows Ports & Adapters applied end-to-end, including a gRPC input adapter alongside REST.

---

> For patterns and architectural concepts (drift detection, legacy modernization, multi-agent orchestration), see [`../patterns/`](../patterns/).
