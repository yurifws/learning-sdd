# AGENTS.md — AI Onboarding Packet
# Task Manager API (Java / Layered)

> This file is written FOR the AI agent, not for humans.
> Read this entirely before writing or modifying any code.
> If anything in this file conflicts with another file, this file wins — flag the discrepancy.

---

## 1. Project Overview

**Project:** Task Manager API — a REST API for creating, reading, updating, and completing tasks with per-user ownership. Demonstrates SDD applied to a standard layered Spring Boot project. Good starting point for CRUD APIs with JWT auth, PostgreSQL, and JUnit integration tests.

**Architecture:** Layered — Controller → Service → Repository

**Language & Runtime:** Java 17+

**Framework:** Spring Boot 3.x

---

## 2. Commands

| Task | Command |
|---|---|
| Install dependencies | `./mvnw dependency:resolve` |
| Start dev server | `./mvnw spring-boot:run` |
| Run all tests | `./mvnw test` |
| Run single test | `./mvnw test -Dtest=TaskServiceTest` |
| Lint / checkstyle | `./mvnw checkstyle:check` |
| Format code | (IDE formatter — see `.editorconfig`) |
| Type check | (compile-time — `./mvnw compile`) |
| Build / package | `./mvnw package` |

> Always use `./mvnw` — never a globally installed Maven.

---

## 3. Project Map

```
task-manager/
├── src/main/java/com/example/tasks/
│   ├── controller/          # HTTP route handlers — thin, no business logic
│   ├── service/             # All business logic lives here
│   ├── repository/          # Spring Data JPA repositories
│   ├── model/               # JPA entity classes
│   ├── dto/                 # Request and response DTOs
│   ├── exception/           # Custom exception classes + global handler
│   └── config/              # Spring Boot configuration (security, beans)
├── src/test/java/           # Mirror the main source structure
│   ├── controller/          # MockMvc controller tests
│   ├── service/             # Unit tests with Mockito
│   └── repository/          # Integration tests with @DataJpaTest
├── src/main/resources/
│   └── db/migration/        # Flyway migration scripts
├── pom.xml
├── CONSTITUTION.md
├── AGENTS.md                # This file
├── CLARIFICATION_GATE.md
├── LIVING_SPEC.md
└── specs/
    └── active/
        └── FEAT-001/        # Task CRUD feature
            ├── requirements.md
            ├── design.md
            ├── tasks.md
            └── TEST_FIRST_GATE.md
```

---

## 4. Stack & Conventions

**Naming conventions:**

| Thing | Convention | Example |
|---|---|---|
| Classes / interfaces | PascalCase | `TaskService`, `TaskController` |
| Methods | camelCase | `findTaskById`, `createTask` |
| Files | PascalCase (Java convention) | `TaskService.java` |
| DB tables | snake_case | `tasks`, `users` |
| DB columns | snake_case | `created_at`, `owner_id` |
| DTOs | PascalCase + `Request`/`Response` | `CreateTaskRequest`, `TaskResponse` |
| Exceptions | PascalCase + `Exception` | `TaskNotFoundException` |

**Code style rules:**

- Controllers handle HTTP ONLY — parse request, delegate to service, return response. No business logic.
- Business logic lives in the service layer only.
- Services never access the HTTP request — they receive domain objects from controllers.
- No `@Autowired` field injection — use constructor injection.
- No `Optional.get()` without a prior `isPresent()` check — use `orElseThrow()`.
- Flyway migrations are the only way to change the schema.

**Approved libraries / dependencies:**

| Allowed | Forbidden |
|---|---|
| Spring Boot 3.x — framework | Quarkus, Micronaut |
| Spring Data JPA — persistence | Plain JDBC in services |
| Flyway — DB migrations | Manual SQL schema changes |
| Lombok — boilerplate reduction | Hand-written getters/setters |
| Spring Security + JWT — auth | Custom auth filters without Spring Security |
| JUnit 5 + Mockito — testing | TestNG, EasyMock |
| Any new dependency | Requires human approval before adding |

---

## 5. Git Workflow

- Branch naming: `feat/short-description`, `fix/short-description`
- Commit format: `feat: add create-task endpoint`
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
- Follow the existing code style of the file you're editing

### Ask before doing
- Adding any new Maven dependency to `pom.xml`
- Changing the database schema (even "small" column additions)
- Modifying existing API response shapes
- Refactoring more than the files listed in the current task
- Introducing a new layer or cross-cutting concern

### Never do
- Put business logic in a controller
- Call a repository directly from a controller
- Bypass JWT authentication on a protected endpoint
- Log passwords, tokens, or user PII
- Push directly to `main`
- Skip a failing test to make a build pass

---

## 7. Testing Expectations

- Every service method must have unit tests using Mockito to mock the repository
- Every controller must have MockMvc tests covering 200, 400, 401, 404 responses
- Every repository must have at least one `@DataJpaTest` integration test
- Test names: `should[ExpectedBehavior]When[Condition]`
- Test files mirror the source structure under `src/test/`
- No real HTTP calls in unit tests — use MockMvc or mock the service

---

## 8. Architecture Rules

- Dependencies flow **downward only**: Controller → Service → Repository
- Controllers never import repositories
- Services never import controllers
- A service method should do one thing — extract helpers for complex logic
- Exception handling is centralized in a `@ControllerAdvice` class — never catch-and-swallow in controllers

For the full rule set, see `CONSTITUTION.md`.

---

## Notes for the AI

- When in doubt, ask. A short clarifying question is always better than a wrong assumption.
- If a task touches authentication, authorization, or user data — stop and confirm before proceeding.
- Prefer small, focused changes. Never refactor outside the current task scope.
- This file is the source of truth for operational context. `CONSTITUTION.md` is the source of truth for rules. If they conflict, flag it.
