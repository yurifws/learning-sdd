# api-contract.md — Application Contract

> **Feature:** Phase 1 — Core Task API
> **Produced by:** Use Case Specialist (Tasks 09–10)
> **Consumed by:** Controller Specialist (Tasks 11–13)
> **Status:** LOCKED — Controller Specialist has started. Any change to this file
> requires stopping all downstream work and re-coordinating with the orchestrator.

---

## What This File Is

This is the contract between the application layer and the controller layer.

The Use Case Specialist produces it. The Controller Specialist reads it — and builds against it — without looking at `TaskUseCase.java`, `TaskPersistencePortOut.java`, or any persistence file. The port-in interface and DTO shapes defined here are the only crossing point.

If the Controller needs something that isn't in this contract, it must stop and flag `[NEEDS_CLARIFICATION]`.

---

## 1. Port-In Interface

### `TaskPortIn`

The only interface the Controller may call. The Controller imports `TaskPortIn` and the request/response records defined in Section 2. Nothing from `application.service` or `application.ports.out`.

```java
package com.example.tasks.application.ports.in;

public interface TaskPortIn {

    /**
     * Creates a new task with status TODO.
     * Throws ValidationException if title is blank or exceeds 255 chars.
     */
    TaskResponse create(CreateTaskRequest request);

    /**
     * Returns the task with the given id.
     * Throws ResourceNotFoundException (→ 404) if not found.
     * DELETED tasks are returned — the caller decides whether to expose them.
     * (Controller must NOT expose DELETED tasks — return 404 if status is DELETED.)
     */
    TaskResponse findById(Long id);

    /**
     * Returns a page of non-DELETED tasks.
     * If filter is non-null, returns only tasks matching that status.
     * page is zero-indexed. size must be between 1 and 100.
     * Throws ValidationException if size > 100.
     */
    PagedResponse<TaskResponse> findAll(int page, int size, TaskStatus filter);

    /**
     * Transitions the task's status to `next`.
     * Throws ResourceNotFoundException (→ 404) if task not found.
     * Throws InvalidStatusTransitionException (→ 422) if transition is not allowed.
     */
    TaskResponse transitionStatus(Long id, TaskStatus next);

    /**
     * Soft-deletes the task by setting status to DELETED.
     * Throws ResourceNotFoundException (→ 404) if task not found.
     * Returns the confirmation message — controller maps this to HTTP 200.
     */
    String softDelete(Long id);
}
```

---

## 2. Request and Response Shapes

### `CreateTaskRequest`

```java
public record CreateTaskRequest(
    @NotBlank(message = "title is required")
    @Size(max = 255, message = "title must be at most 255 characters")
    String title,

    String description,           // nullable

    Long assigneeId,              // nullable

    @JsonFormat(pattern = "yyyy-MM-dd")
    LocalDate dueDate             // nullable
) {}
```

### `TransitionStatusRequest`

```java
public record TransitionStatusRequest(
    @NotNull(message = "status is required")
    TaskStatus status             // TODO | IN_PROGRESS | DONE (never DELETED — use DELETE endpoint)
) {}
```

### `TaskResponse`

Returned by all endpoints that return a single task.

```java
public record TaskResponse(
    Long id,
    String title,
    String description,          // null if not set
    TaskStatus status,
    Long assigneeId,             // null if not set
    @JsonFormat(pattern = "yyyy-MM-dd")
    LocalDate dueDate,           // null if not set
    @JsonFormat(pattern = "yyyy-MM-dd'T'HH:mm:ss")
    LocalDateTime createdAt,
    @JsonFormat(pattern = "yyyy-MM-dd'T'HH:mm:ss")
    LocalDateTime updatedAt
) {}
```

### `PagedResponse<T>`

Returned by the list endpoint.

```java
public record PagedResponse<T>(
    List<T> content,
    int page,
    int size,
    long totalElements,
    int totalPages
) {}
```

---

## 3. Endpoint Contract

| Method | Path | Auth | Request body | Success | Error cases |
|---|---|---|---|---|---|
| `POST` | `/api/v1/tasks` | `ROLE_ADMIN` | `CreateTaskRequest` | `201 TaskResponse` | `400` validation, `401` missing JWT, `403` wrong role |
| `GET` | `/api/v1/tasks/{id}` | any role | — | `200 TaskResponse` | `404` not found or DELETED, `401` |
| `GET` | `/api/v1/tasks` | any role | query params: `page`, `size`, `status` | `200 PagedResponse<TaskResponse>` | `400` size > 100, `401` |
| `PATCH` | `/api/v1/tasks/{id}/status` | `ROLE_ADMIN` | `TransitionStatusRequest` | `200 TaskResponse` | `404`, `422` invalid transition, `401`, `403` |
| `DELETE` | `/api/v1/tasks/{id}` | `ROLE_ADMIN` | — | `200 { "message": "Task deleted successfully" }` | `404`, `401`, `403` |

**Auth rule:** `@PreAuthorize` is placed on the `TaskOpenApi` interface, not on `TaskController`. This is a non-negotiable rule from `CONSTITUTION.md`.

---

## 4. Error Response Shape

All error responses follow this structure — no raw stack traces, no Spring default error page.

```json
{
  "error": "ERROR_CODE",
  "message": "Human-readable description"
}
```

| Scenario | HTTP status | `error` code |
|---|---|---|
| Bean validation failure | `400` | `VALIDATION_ERROR` |
| Task not found | `404` | `NOT_FOUND` |
| Invalid status transition | `422` | `INVALID_TRANSITION` |
| Missing or expired JWT | `401` | `UNAUTHORIZED` |
| Insufficient role | `403` | `FORBIDDEN` |

---

## 5. Status Transition Rules

The Controller does not enforce these — they are enforced in `TaskDomain.transitionTo()` and the Use Case. The Controller only maps `InvalidStatusTransitionException` to HTTP 422.

```
TODO        → IN_PROGRESS  ✓
IN_PROGRESS → DONE         ✓
IN_PROGRESS → TODO         ✓
DONE        → (nothing)    ✗  all transitions from DONE are forbidden
DELETED     → (nothing)    ✗  all transitions from DELETED are forbidden
```

`TransitionStatusRequest` must not accept `DELETED` as a value — soft delete is done via the DELETE endpoint only.

---

## 6. Gate C Sign-Off

> Reviewed by: [human]
> Date: 2025-01-01
> Status: **PASSED**

Checklist:
- [x] Port-in methods cover all 5 endpoints from `design.md §5`
- [x] All DTO shapes match `design.md §4` exactly
- [x] Error codes and HTTP status mapping is defined
- [x] Status transition rules are documented (Controller does not re-implement them)
- [x] Auth rules follow `CONSTITUTION.md` (`@PreAuthorize` on interface)
- [x] `./mvnw test -Dtest=TaskUseCaseTest` — all tests green

**This contract is now LOCKED. The Controller Specialist may begin.**
