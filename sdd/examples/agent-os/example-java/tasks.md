# tasks.md — Task Management API

> **Feature:** Phase 1 — Core Task API
> **Linked:** requirements.md · design.md · CONSTITUTION.md
> **Date:** 2025-01-01

---

## Instructions for Claude

Read `AGENTS.md` first, then `CONSTITUTION.md`, `CLARIFICATION_GATE.md`, `LIVING_SPEC.md`, and this file before starting.
Implement one task at a time. Stop after each task and wait for approval.
If anything is ambiguous, flag it as `[NEEDS_CLARIFICATION]` and stop.

---

## Phase 1 — Foundation

- [ ] **TASK-01** — Create Maven project with Spring Boot 3.3, Java 21.
  Add dependencies: `spring-boot-starter-web`, `spring-boot-starter-data-jpa`,
  `spring-boot-starter-security`, `spring-boot-starter-validation`,
  `postgresql`, `flyway-core`, `jjwt-api`, `mapstruct`, `springdoc-openapi-starter-webmvc-ui`.
  **Verify:** `./mvnw clean install` passes with no errors.

- [ ] **TASK-02** — Create `application.yml` with DB config, JWT public-key placeholder,
  and `api.base-path=/api/v1`.
  **Verify:** App starts without missing property errors.

- [ ] **TASK-03** — Create Flyway migration V1: `tasks` table (design.md Section 3).
  **Verify:** App starts and Flyway applies V1 cleanly — no schema errors.

- [ ] **TASK-04** — Create `TaskStatus` enum: `TODO`, `IN_PROGRESS`, `DONE`, `DELETED`.
  Create `TaskDomain` with `transitionTo(TaskStatus)` method and valid-transition guard.
  **Verify:** Unit test `TaskDomainTest.validTransitions_pass` and `invalidTransitions_throw` both pass.

- [ ] **TASK-05** — Create `GlobalExceptionHandler` with mappings for all exceptions
  (design.md Section 7). Never expose stack traces.
  **Verify:** Throwing `ResourceNotFoundException` in a test controller returns
  `{ "error": "NOT_FOUND", "message": "..." }`.

---

## Phase 2 — Persistence Layer

- [ ] **TASK-06** — Create `TaskEntity` (JPA) and `TaskRepository` extending `JpaRepository<TaskEntity, Long>`.
  Add method: `Page<TaskEntity> findAllByStatusNot(TaskStatus status, Pageable pageable)`.
  **Verify:** App starts; Hibernate validates schema.

- [ ] **TASK-07** — Create `TaskPersistenceMapper` (MapStruct): `TaskEntity ↔ TaskDomain`.
  **Verify:** Project compiles; no manual field assignments.

- [ ] **TASK-08** — Create `TaskPersistencePortOut` interface with methods:
  `save(TaskDomain)`, `findById(Long)`, `findAll(Pageable, TaskStatus filter)`, `delete(Long)`.
  Create `TaskPersistenceService` implementing `TaskPersistencePortOut`.
  **Verify:** Unit test `TaskPersistenceServiceTest.findById_unknownId_throwsNotFoundException` passes.

---

## Phase 3 — Application Layer

- [ ] **TASK-09** — Create `TaskPortIn` interface with methods:
  `create(CreateTaskRequest)`, `findById(Long)`, `findAll(int page, int size, TaskStatus filter)`,
  `transitionStatus(Long id, TaskStatus next)`, `softDelete(Long id)`.
  **Verify:** Project compiles.

- [ ] **TASK-10** — Implement `TaskUseCase` implementing `TaskPortIn`.
  - `create`: build `TaskDomain` with status TODO → call `portOut.save`
  - `findById`: call `portOut.findById` or throw `ResourceNotFoundException`
  - `findAll`: validate size ≤ 100, call `portOut.findAll`, exclude DELETED
  - `transitionStatus`: load task → call `domain.transitionTo` → save
  - `softDelete`: load task → set status DELETED → save
  **Verify:** `./mvnw test -Dtest=TaskUseCaseTest` passes (includes T9 from TEST_FIRST_GATE).

---

## Phase 4 — Controller Layer

- [ ] **TASK-11** — Create `CreateTaskRequest`, `TaskResponse`, `TransitionStatusRequest`
  records with Bean Validation annotations matching requirements.md §2.2.
  **Verify:** Project compiles; `@NotBlank` present on `title`.

- [ ] **TASK-12** — Create `TaskControllerMapper` (MapStruct): `TaskDomain ↔ TaskResponse`.
  **Verify:** Project compiles.

- [ ] **TASK-13** — Create `TaskOpenApi` interface and `TaskController` with all 5 endpoints
  (design.md Section 5). Add `@PreAuthorize` on every method.
  Return `ResponseEntity<TaskResponse>` or `ResponseEntity<PagedResponse<TaskResponse>>`.
  **Verify:** `./mvnw test -Dtest=TaskControllerTest` passes (T1–T8 from TEST_FIRST_GATE).

---

## Phase 5 — Security

- [ ] **TASK-14** — Create `JwtUtil` with `parseToken(String jwt)` and `getClaims(String jwt)`.
  **Verify:** Unit test with a known RS256 JWT passes.

- [ ] **TASK-15** — Create `JwtAuthenticationFilter` extending `OncePerRequestFilter`.
  **Verify:** Project compiles.

- [ ] **TASK-16** — Configure `SecurityFilterChain`:
  - Require JWT on all routes.
  - Allow unauthenticated access to POST `/api/v1/auth/**`.
  - `ROLE_ADMIN` required for POST, PATCH, DELETE on `/api/v1/tasks`.
  **Verify:** Request without token to POST /api/v1/tasks returns HTTP 401.

---

## Phase 6 — Full Suite

- [ ] **TASK-17** — Run all tests with Testcontainers (integration suite).
  **Verify:** `./mvnw verify` — zero failures, coverage ≥ 80% on `TaskUseCase`.

---

## Claude Prompt Templates

### Start a task
```
Read AGENTS.md, CONSTITUTION.md, CLARIFICATION_GATE.md, LIVING_SPEC.md,
requirements.md, design.md, and tasks.md before starting.
Implement TASK-04: create TaskStatus enum and TaskDomain with transitionTo().
Do not implement TASK-05 yet.
If anything is ambiguous, flag it [NEEDS_CLARIFICATION] and stop.
```

### Continue after approval
```
TASK-04 approved. Now implement TASK-05.
Re-read design.md Section 7 before starting.
```

### After a test failure
```
This test is failing: [paste error]
Before guessing, re-read CONSTITUTION.md and check the domain transition rules in design.md Section 6.
Fix the issue. Do not change the test unless the requirement changed.
```

### Full project kickoff
```
I want to build a Task Management REST API.
Read AGENTS.md, CONSTITUTION.md, CLARIFICATION_GATE.md, LIVING_SPEC.md,
requirements.md, design.md, and tasks.md before starting.
Follow CONSTITUTION.md at all times — no exceptions.
Start with TASK-01. After each task, summarize what you built and wait for my approval.
```
