# db-contract.md — Persistence Contract

> **Feature:** Phase 1 — Core Task API
> **Produced by:** Persistence Specialist (Tasks 06–08)
> **Consumed by:** Use Case Specialist (Tasks 09–10)
> **Status:** LOCKED — Use Case Specialist has started. Any change to this file
> requires stopping all downstream work and re-coordinating with the orchestrator.

---

## What This File Is

This is the contract between the persistence layer and the application layer.

The Persistence Specialist produces it. The Use Case Specialist reads it — and builds against it — without looking at `TaskEntity.java`, `TaskRepository.java`, or any other persistence file. The port interface defined here is the only crossing point between layers.

If the Use Case needs something that isn't in this contract, it must stop and flag `[NEEDS_CLARIFICATION]`. It must not reach into the persistence layer directly.

---

## 1. Database Schema

### Table: `tasks`

Flyway migration: `V1__create_task_table.sql`

| Column | Type | Constraints | Notes |
|---|---|---|---|
| `id` | `BIGSERIAL` | `PRIMARY KEY` | Auto-generated |
| `title` | `VARCHAR(255)` | `NOT NULL` | Max 255 chars enforced at DB level |
| `description` | `TEXT` | nullable | |
| `status` | `VARCHAR(20)` | `NOT NULL`, `DEFAULT 'TODO'` | Values: `TODO`, `IN_PROGRESS`, `DONE`, `DELETED` |
| `assignee_id` | `BIGINT` | nullable | FK → `users(id)` — no cascade, FK not enforced in Phase 1 |
| `due_date` | `DATE` | nullable | |
| `created_at` | `TIMESTAMP` | `NOT NULL`, `DEFAULT NOW()` | Set once on insert |
| `updated_at` | `TIMESTAMP` | `NOT NULL` | Updated via `@PreUpdate` on entity |

**Indexes:**
- `idx_tasks_status` on `(status)` — list queries filter by status
- Primary key index on `id`

**No unique constraints** — duplicate titles are allowed in Phase 1.

---

## 2. Port-Out Interface

### `TaskPersistencePortOut`

This is the only interface the Use Case layer may call. The Use Case must import this interface and nothing from the persistence package.

```java
package com.example.tasks.application.ports.out;

public interface TaskPersistencePortOut {

    /**
     * Persists a new task. The domain object must have status = TODO.
     * Returns the saved domain object with the generated id populated.
     */
    TaskDomain save(TaskDomain task);

    /**
     * Returns the task with the given id.
     * Throws ResourceNotFoundException if no task with that id exists.
     * DELETED tasks are returned by this method — filtering is the Use Case's responsibility.
     */
    Optional<TaskDomain> findById(Long id);

    /**
     * Returns a page of tasks excluding DELETED status.
     * If filter is non-null, returns only tasks matching that status.
     * Page is zero-indexed.
     */
    Page<TaskDomain> findAll(Pageable pageable, TaskStatus filter);

    /**
     * Persists an updated task (status transition or field change).
     * The task must already exist — throws ResourceNotFoundException if id not found.
     */
    TaskDomain update(TaskDomain task);
}
```

**What this interface does NOT include:**
- No `delete(Long id)` — soft delete is implemented in the Use Case by calling `update()` with status DELETED
- No `findByStatus()` — the `filter` parameter on `findAll()` covers this
- No batch operations — Phase 1 is single-entity only

---

## 3. Domain Object Expected by This Layer

The Persistence Specialist maps between `TaskEntity` (JPA) and `TaskDomain` (domain). The Use Case layer works exclusively with `TaskDomain`.

`TaskDomain` fields visible to the Use Case (produced by Domain Specialist, Task 04):

```java
public class TaskDomain {
    Long id;                // null on creation, populated after save()
    String title;           // required
    String description;     // nullable
    TaskStatus status;      // see transition rules in design.md §6
    Long assigneeId;        // nullable
    LocalDate dueDate;      // nullable
    LocalDateTime createdAt;
    LocalDateTime updatedAt;

    // Business method — not a setter
    public TaskDomain transitionTo(TaskStatus next); // throws InvalidStatusTransitionException
}
```

---

## 4. Exception Contract

The persistence layer throws these exceptions. The Use Case layer must handle or propagate them — it must not catch and swallow them silently.

| Exception | When thrown | Who handles it |
|---|---|---|
| `ResourceNotFoundException` | `findById()` with unknown id | Propagated to `GlobalExceptionHandler` → HTTP 404 |
| `ResourceNotFoundException` | `update()` with unknown id | Same |
| `DataIntegrityViolationException` | DB constraint violated (e.g. null title) | Propagated — Bean Validation at controller layer prevents this in normal flow |

---

## 5. Gate B Sign-Off

> Reviewed by: [human]
> Date: 2025-01-01
> Status: **PASSED**

Checklist:
- [x] Table schema matches `design.md §3` exactly
- [x] Port-out method signatures cover all Use Case operations
- [x] No business logic in the persistence layer (only mapping and JPA calls)
- [x] Exception types are defined and documented
- [x] `./mvnw test -Dtest=TaskPersistenceServiceTest` — all tests green

**This contract is now LOCKED. The Use Case Specialist may begin.**
