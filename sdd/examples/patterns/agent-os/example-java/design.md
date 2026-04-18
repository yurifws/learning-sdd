# design.md — Task Management API Design

> **Feature:** Phase 1 — Core Task API
> **Linked:** requirements.md
> **Date:** 2025-01-01

---

## 1. Architecture

```
HTTP Request
    │
    ▼
TaskController                ← HTTP adapter: parses request, calls PortIn
    │
    ▼
TaskPortIn (interface)        ← input port: what the outside world calls
    │
    ▼
TaskUseCase                   ← business logic, status transition validation
    │
    ▼
TaskPersistencePortOut        ← output port: what the app needs from outside
    │
    ▼
TaskPersistenceService        ← JPA calls, NotFoundException on missing rows
    │
    ▼
TaskRepository                ← Spring Data JPA
    │
    ▼
PostgreSQL (tasks table)
```

Status transition rules enforced in `TaskDomain.transitionTo(TaskStatus next)`.

---

## 2. Package Structure

```
com.example.tasks
├── adapters/
│   ├── input/
│   │   └── controller/
│   │       ├── TaskController.java
│   │       ├── TaskOpenApi.java
│   │       └── mapper/
│   │           └── TaskControllerMapper.java
│   └── output/
│       └── persistence/
│           ├── TaskPersistenceService.java
│           ├── entity/
│           │   └── TaskEntity.java
│           ├── repository/
│           │   └── TaskRepository.java
│           └── mapper/
│               └── TaskPersistenceMapper.java
├── application/
│   ├── ports/
│   │   ├── in/
│   │   │   └── TaskPortIn.java
│   │   └── out/
│   │       └── TaskPersistencePortOut.java
│   └── service/
│       └── TaskUseCase.java
├── domain/
│   ├── TaskDomain.java
│   └── TaskStatus.java
├── config/
│   └── SecurityConfig.java
└── exception/
    ├── ResourceNotFoundException.java
    ├── InvalidStatusTransitionException.java
    └── GlobalExceptionHandler.java
```

---

## 3. Data Model

### Entity: `tasks` table (Flyway migration V1)

| Column | Type | Constraints |
|---|---|---|
| id | BIGSERIAL | PK |
| title | VARCHAR(255) | NOT NULL |
| description | TEXT | nullable |
| status | VARCHAR(20) | NOT NULL, DEFAULT 'TODO' |
| assignee_id | BIGINT | nullable, FK → users(id) |
| due_date | DATE | nullable |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() |
| updated_at | TIMESTAMP | NOT NULL, auto-set via `@PreUpdate` |

---

## 4. DTOs

### CreateTaskRequest (POST body)
```json
{
  "title": "string, required, max 255",
  "description": "string, optional",
  "assigneeId": "long, optional",
  "dueDate": "date, optional, ISO-8601 (yyyy-MM-dd)"
}
```

### TaskResponse (all endpoints)
```json
{
  "id": "long",
  "title": "string",
  "description": "string | null",
  "status": "TODO | IN_PROGRESS | DONE | DELETED",
  "assigneeId": "long | null",
  "dueDate": "yyyy-MM-dd | null",
  "createdAt": "ISO-8601 timestamp",
  "updatedAt": "ISO-8601 timestamp"
}
```

### TransitionStatusRequest (PATCH body)
```json
{
  "status": "TODO | IN_PROGRESS | DONE"
}
```

### PagedResponse<T> (list endpoint)
```json
{
  "content": "T[]",
  "page": "integer",
  "size": "integer",
  "totalElements": "long",
  "totalPages": "integer"
}
```

---

## 5. API Endpoints

| Method | Path | Auth | Description |
|---|---|---|---|
| POST | /api/v1/tasks | ROLE_ADMIN | Create task |
| GET | /api/v1/tasks | Any role | List tasks (paginated) |
| GET | /api/v1/tasks/{id} | Any role | Get task by ID |
| PATCH | /api/v1/tasks/{id}/status | ROLE_ADMIN | Transition status |
| DELETE | /api/v1/tasks/{id} | ROLE_ADMIN | Soft delete |

---

## 6. Status Transition Logic (Domain Layer)

```java
public class TaskDomain {
    // ...
    public void transitionTo(TaskStatus next) {
        if (!isValidTransition(this.status, next)) {
            throw new InvalidStatusTransitionException(
                "Cannot transition from " + this.status + " to " + next
            );
        }
        this.status = next;
    }

    private static boolean isValidTransition(TaskStatus from, TaskStatus to) {
        return switch (from) {
            case TODO         -> to == TaskStatus.IN_PROGRESS;
            case IN_PROGRESS  -> to == TaskStatus.DONE || to == TaskStatus.TODO;
            default           -> false;  // DONE, DELETED are terminal
        };
    }
}
```

---

## 7. Exception Handling

| Exception | HTTP | Code |
|---|---|---|
| `ResourceNotFoundException` | 404 | NOT_FOUND |
| `InvalidStatusTransitionException` | 422 | INVALID_TRANSITION |
| `MethodArgumentNotValidException` | 400 | VALIDATION_ERROR |
| `AccessDeniedException` | 403 | FORBIDDEN |
| `Exception` (fallback) | 500 | INTERNAL_ERROR |

---

## 8. Technical Decisions

| Decision | Choice | Reason |
|---|---|---|
| Status transitions in domain | `TaskDomain.transitionTo()` | Domain logic belongs in the domain, not in controllers |
| Soft delete | Set `status = DELETED` | Preserve data for audit log (Phase 3) |
| MapStruct | All mappers | Type-safe, compile-time checked, no manual builders |
| Testcontainers | Integration tests | Tests hit a real PostgreSQL — no in-memory divergence |
| Java records for DTOs | `CreateTaskRequest`, `TaskResponse` | Immutable, no Lombok |
| RS256 JWT | Public/private key pair | More secure than symmetric keys; keys rotatable independently |
