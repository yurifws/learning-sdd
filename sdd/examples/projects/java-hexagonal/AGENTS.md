# AGENTS.md — AI Onboarding Packet
# Client Management API (Java / Hexagonal)

> This file is written FOR the AI agent, not for humans.
> Read this entirely before writing or modifying any code.
> If anything in this file conflicts with another file, this file wins — flag the discrepancy.

---

## 1. Project Overview

**Project:** Client Management API — a REST + gRPC API for managing client records (summary, addresses, contacts, plan). Built as a reference implementation demonstrating Hexagonal Architecture (Ports & Adapters) with Spring Boot. Demonstrates enterprise patterns: Oracle-style table naming, `@PreAuthorize` security, MapStruct mapping, OpenAPI documentation.

**Architecture:** Hexagonal (Ports & Adapters)

**Language & Runtime:** Java 17+

**Framework:** Spring Boot 3.x

---

## 2. Commands

| Task | Command |
|---|---|
| Install dependencies | `./mvnw dependency:resolve` |
| Start dev server | `./mvnw spring-boot:run` |
| Run all tests | `./mvnw test` |
| Run single test | `./mvnw test -Dtest=ClientUseCaseTest` |
| Lint / checkstyle | `./mvnw checkstyle:check` |
| Format code | (IDE formatter — see `.editorconfig`) |
| Type check | (compile-time — `./mvnw compile`) |
| Build / package | `./mvnw package` |

> Always use `./mvnw` — never a globally installed Maven.

---

## 3. Project Map

```
tech.example.api.client/
├── src/main/java/tech/example/api/client/
│   ├── adapters/
│   │   ├── input/
│   │   │   ├── controller/      # REST controllers — HTTP only, no business logic
│   │   │   └── grpc/            # gRPC input adapter (optional)
│   │   └── output/
│   │       └── persistence/     # JPA repositories + persistence services
│   ├── application/
│   │   ├── ports/
│   │   │   ├── in/              # Input port interfaces (PortIn) — called by adapters
│   │   │   └── out/             # Output port interfaces (PortOut) — implemented by persistence
│   │   └── service/             # Use cases — ALL business logic lives here
│   ├── domain/                  # Plain Java POJOs — ZERO framework annotations
│   └── config/                  # Spring Boot configuration classes
├── src/test/java/               # Mirror the main source structure
├── pom.xml
├── CONSTITUTION.md
├── AGENTS.md                    # This file
├── CLARIFICATION_GATE.md
├── requirements.md              # EARS requirements for all endpoints
├── design.md                    # Architecture, package structure, data model, examples
└── tasks.md                     # Ordered task list
```

---

## 4. Stack & Conventions

**Naming conventions:**

| Thing | Convention | Example |
|---|---|---|
| Classes / interfaces | PascalCase | `ClientUseCase`, `ClientPersistenceService` |
| Methods | camelCase | `findClientSummaryById` |
| Files | PascalCase (Java convention) | `ClientController.java` |
| DB tables | `T_TABLENAME` (Oracle-style) | `T_CLIENT`, `T_ADDRESS` |
| DB columns | `IDCOLUMN` / `COLUMNNAME` | `IDCLIENT`, `NAMECLIENT` |
| PortIn interfaces | `[Feature]UseCase` | `ClientUseCase` |
| PortOut interfaces | `[Feature]Port` | `ClientPort` |
| Persistence services | `[Feature]PersistenceService` | `ClientPersistenceService` |

**Code style rules:**

- Controllers handle HTTP ONLY — parse request, call PortIn, return response. No business logic.
- Use cases implement PortIn and only call PortOut interfaces — NEVER repositories directly.
- Persistence services implement PortOut and call repositories.
- Domain objects are pure Java — zero JPA or Spring annotations.
- Response models use Java Records — immutable, concise.
- Security declarations (`@PreAuthorize`) go on the OpenAPI interface, not the implementation.
- Native queries (`nativeQuery = true`) used where JPQL cannot express complex joins.

**Approved libraries / dependencies:**

| Allowed | Forbidden |
|---|---|
| Spring Boot 3.x — framework | Quarkus, Micronaut |
| Spring Data JPA — persistence | Plain JDBC in services |
| MapStruct — mapping | Manual mapping code, ModelMapper |
| Lombok — boilerplate reduction | Hand-written getters/setters |
| OpenAPI 3 / SpringDoc — docs | (no Swagger 2) |
| Spring Security — auth | Custom auth filters |
| Any new dependency | Requires human approval before adding |

---

## 5. Git Workflow

- Branch naming: `feat/short-description`, `fix/short-description`
- Commit format: `feat: add get-client-summary endpoint`
- Spec commits first: `spec: update requirements for [feature]`, then `plan: ...`, then `feat: ...`
- Always branch from `main`
- Never force-push to `main`
- PRs must pass `./mvnw test` before merging

---

## 6. Boundaries — Safety Rules

### Always do
- Re-read `CONSTITUTION.md` before starting each task — follow every rule without exception
- Follow spec-first order: update requirements → update design/tasks → implement → verify
- Run `./mvnw test` before marking any task done
- Add or update tests when changing any business logic
- Keep the layer boundaries strict: Controller → PortIn → UseCase → PortOut → Persistence

### Ask before doing
- Adding any new Maven dependency to `pom.xml`
- Changing the database schema
- Modifying API response shapes
- Refactoring more than the files listed in the current task
- Introducing a new layer or design pattern

### Never do
- Call a JPA repository from a use case or controller
- Put business logic in a controller
- Put JPA/Spring annotations on domain objects
- Bypass `@PreAuthorize` security rules
- Push directly to `main`
- Skip a failing test to make a build pass

---

## 7. Testing Expectations

- Every use case must have unit tests in `application/service/`
- Every persistence service must have integration tests hitting H2
- Every controller must have MockMvc tests covering happy path and error responses
- Test names: `should[ExpectedBehavior]When[Condition]`
- Test files mirror the source structure under `src/test/`
- No real external calls in unit tests — mock PortOut

---

## 8. Architecture Rules

- Dependencies flow **inward**: Controller → PortIn (interface) → UseCase → PortOut (interface) → Persistence
- The domain layer has **zero knowledge** of any other layer
- A use case may only interact with the outside world through PortOut interfaces
- Adding a new adapter (e.g., Kafka consumer) requires a new PortIn implementation — the use case is untouched
- Every new endpoint requires a corresponding OpenAPI annotation

For the full rule set and golden rules, see `CONSTITUTION.md`.

---

## Notes for the AI

- When in doubt, ask. A short clarifying question is always better than a wrong assumption.
- If a task touches authentication, authorization, or data access — stop and confirm before proceeding.
- Prefer small, focused changes. Never refactor outside the current task scope.
- This file is the source of truth for operational context. `CONSTITUTION.md` is the source of truth for rules. If they conflict, flag it.
