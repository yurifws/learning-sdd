# requirements.md — Task Management API Requirements

> **Feature:** Phase 1 — Core Task API
> **Date:** 2025-01-01
> **Status:** Approved
> **Linked:** CLARIFICATION_GATE.md · design.md · tasks.md

---

## 1. Overview

**Goal:** Expose authenticated REST endpoints to create, read, update, and soft-delete tasks.
**Out of scope:** Assignment (Phase 2), audit log (Phase 3), notifications (Phase 4).

---

## 2. Functional Requirements (EARS Notation)

### 2.1 Ubiquitous (Always true)

```
The API SHALL require a valid RS256 JWT Bearer token on every endpoint.
The API SHALL validate all request bodies using Jakarta Bean Validation (@Valid).
The API SHALL return all error responses as structured JSON — never raw stack traces.
The API SHALL NOT include DELETED tasks in any list response.
```

---

### 2.2 Event-Driven — Create Task (POST /tasks)

```
WHEN a POST request is sent to /api/v1/tasks with a valid body,
  IF the caller has ROLE_ADMIN,
  THEN the system SHALL persist the task with status TODO
    AND SHALL return HTTP 201 with the created TaskResponse.

IF title is null or blank,
  THEN the system SHALL return HTTP 400 with error code VALIDATION_ERROR
    AND message "title is required".

IF title exceeds 255 characters,
  THEN the system SHALL return HTTP 400 with error code VALIDATION_ERROR
    AND message "title must be at most 255 characters".
```

---

### 2.3 Event-Driven — Get Task (GET /tasks/{id})

```
WHEN a GET request is sent to /api/v1/tasks/{id},
  IF the task with that ID exists AND its status is NOT DELETED,
  THEN the system SHALL return HTTP 200 with TaskResponse.

  IF the task does not exist OR its status is DELETED,
  THEN the system SHALL return HTTP 404 with error code NOT_FOUND.
```

---

### 2.4 Event-Driven — List Tasks (GET /tasks)

```
WHEN a GET request is sent to /api/v1/tasks,
  the system SHALL return a paginated list of tasks with status != DELETED.
  The system SHALL support query params:
    - status (optional): filter by TODO | IN_PROGRESS | DONE
    - page (default 0)
    - size (default 20, max 100)
  The system SHALL sort results by createdAt DESC by default.

IF size exceeds 100,
  THEN the system SHALL return HTTP 400 with error code VALIDATION_ERROR
    AND message "page size must not exceed 100".
```

---

### 2.5 Event-Driven — Transition Status (PATCH /tasks/{id}/status)

```
WHEN a PATCH request is sent to /api/v1/tasks/{id}/status,
  IF the caller has ROLE_ADMIN
    AND the task exists
    AND the transition is valid (TODO→IN_PROGRESS, IN_PROGRESS→DONE, IN_PROGRESS→TODO),
  THEN the system SHALL update the task status
    AND SHALL return HTTP 200 with the updated TaskResponse.

  IF the transition is invalid,
  THEN the system SHALL return HTTP 422 with error code INVALID_TRANSITION
    AND a message describing the invalid transition.

  IF the task does not exist,
  THEN the system SHALL return HTTP 404.
```

---

### 2.6 Event-Driven — Soft Delete (DELETE /tasks/{id})

```
WHEN a DELETE request is sent to /api/v1/tasks/{id},
  IF the caller has ROLE_ADMIN AND the task exists,
  THEN the system SHALL set task status to DELETED
    AND SHALL NOT remove the row from the database
    AND SHALL return HTTP 200 with message "Task deleted successfully".

  IF the task does not exist,
  THEN the system SHALL return HTTP 404.
```

---

### 2.7 Unwanted Behavior (Security Rules)

```
The system SHALL NOT allow ROLE_USER to create, update, or delete tasks.
The system SHALL NOT expose stack traces in any response body.
The system SHALL NOT return tasks with status DELETED in any list or get-by-id response.
IF a request is made without a valid JWT,
  THEN the system SHALL return HTTP 401.
IF a request is made with a valid JWT but insufficient role,
  THEN the system SHALL return HTTP 403.
```

---

## 3. Non-Functional Requirements

| Category | Requirement |
|---|---|
| Performance | GET /tasks must respond in < 300ms for up to 10,000 tasks |
| Security | JWT RS256. ROLE_ADMIN for mutations. ROLE_USER for reads. |
| Pagination | Default 20, max 100. Exceed 100 → 400. |
| Availability | 99.9% uptime |
| Test coverage | ≥ 80% branch coverage on service layer |

---

## 4. Acceptance Criteria

### Create task
- [ ] POST with valid body returns HTTP 201 with TaskResponse
- [ ] POST with blank title returns HTTP 400
- [ ] POST without JWT returns HTTP 401
- [ ] POST with ROLE_USER returns HTTP 403

### Get task
- [ ] GET with valid ID returns HTTP 200
- [ ] GET with unknown ID returns HTTP 404
- [ ] GET with DELETED task ID returns HTTP 404

### List tasks
- [ ] GET /tasks returns paginated list of non-deleted tasks
- [ ] GET /tasks?status=TODO returns only TODO tasks
- [ ] GET /tasks?size=101 returns HTTP 400

### Status transition
- [ ] PATCH TODO→IN_PROGRESS returns HTTP 200
- [ ] PATCH IN_PROGRESS→DONE returns HTTP 200
- [ ] PATCH TODO→DONE returns HTTP 422 with INVALID_TRANSITION
- [ ] PATCH on non-existent task returns HTTP 404

### Soft delete
- [ ] DELETE sets status DELETED, row remains in DB
- [ ] Deleted task does NOT appear in GET /tasks
- [ ] DELETE on non-existent task returns HTTP 404
