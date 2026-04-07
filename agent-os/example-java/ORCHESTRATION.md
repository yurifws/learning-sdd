# ORCHESTRATION.md — Task Management API

> This is a filled-in example. See the blank template at `agent-os/ORCHESTRATION.md`.

---

## Feature Being Orchestrated

**Feature:** FEAT-001 — Create Task endpoint (`POST /tasks`)
**Reason for orchestration:** 4 tasks across independent layers with clean file boundaries.

---

## Orchestrator Instructions

```
You are the orchestrator for FEAT-001 (POST /tasks).

Read in order:
1. PROJECT_PLAN.md       — scope: Task CRUD, JWT auth, no frontend
2. PROJECT_ROADMAP.md    — Phase 1, current priority is Core Task API
3. PROJECT_TECHSTACK.md  — Java 21, Spring Boot 3.3, Hexagonal, no Lombok, MapStruct
4. tasks.md              — 4 tasks: domain → persistence → use case → controller

Your job:
- Delegate Task 1 to the Domain Specialist
- After Task 1 is verified, delegate Task 2 to the Persistence Specialist
- After Task 2 is verified, delegate Task 3 to the Use Case Specialist
- After Task 3 is verified, delegate Task 4 to the Controller Specialist
- After all done, run: ./mvnw verify and confirm it passes
- If any specialist reports a conflict or ambiguity, stop all work and flag [NEEDS_CLARIFICATION]

Note: Tasks 1-2-3-4 are sequential (each layer depends on the one below).
Do NOT run them in parallel.
```

> Why sequential here? The domain object is created in Task 1 and imported by all other layers.
> Parallel execution would cause compile errors. Sequence by dependency first, parallelize the rest.

---

## Task Breakdown

### Task 1 — Domain Object
**Agent role:** Domain Specialist
**Files to touch:**
- `src/main/java/.../domain/Task.java`
- `src/main/java/.../domain/TaskStatus.java`

**Must NOT touch:** Any controller, use case, or persistence file
**Verify:** `./mvnw compile`
**Depends on:** nothing — start here

---

### Task 2 — Persistence Layer
**Agent role:** Persistence Specialist
**Files to touch:**
- `src/main/java/.../adapters/output/persistence/entity/TaskEntity.java`
- `src/main/java/.../adapters/output/persistence/TaskJpaRepository.java`
- `src/main/java/.../adapters/output/persistence/TaskPersistenceService.java`
- `src/main/resources/db/migration/V1__create_task_table.sql`

**Must NOT touch:** Any controller, use case, or domain file
**Verify:** `./mvnw test -Dtest=TaskPersistenceServiceTest`
**Depends on:** Task 1 must be verified before starting

---

### Task 3 — Use Case
**Agent role:** Use Case Specialist
**Files to touch:**
- `src/main/java/.../application/ports/in/CreateTaskUseCase.java`
- `src/main/java/.../application/ports/out/SaveTaskPort.java`
- `src/main/java/.../application/service/CreateTaskService.java`

**Must NOT touch:** Any controller or persistence file
**Verify:** `./mvnw test -Dtest=CreateTaskServiceTest`
**Depends on:** Task 2 must be verified before starting

---

### Task 4 — Controller
**Agent role:** Controller Specialist
**Files to touch:**
- `src/main/java/.../adapters/input/controller/TaskController.java`
- `src/main/java/.../adapters/input/controller/dto/CreateTaskRequest.java`
- `src/main/java/.../adapters/input/controller/dto/TaskResponse.java`
- `src/test/java/.../adapters/input/controller/TaskControllerIT.java`

**Must NOT touch:** Any use case, persistence, or domain file
**Verify:** `./mvnw test -Dtest=TaskControllerIT`
**Depends on:** Task 3 must be verified before starting

---

## Final Verification

After all 4 tasks are done and individually verified:

```bash
./mvnw verify
```

Expected: BUILD SUCCESS, all tests green, no compilation warnings.

---

## Conflict Prevention Checklist

- [x] Every task has an explicit `Files to touch` list
- [x] No file appears in more than one task
- [x] Tasks are sequenced by layer dependency (domain → persistence → use case → controller)
- [x] Each task has its own `Verify` command
- [x] No `[NEEDS_CLARIFICATION]` items remain in tasks.md