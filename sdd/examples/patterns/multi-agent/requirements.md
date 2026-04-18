# requirements.md — Task Dashboard

> **Feature:** Task Dashboard (FEAT-002)
> **Status:** Approved
> **Stack:** PostgreSQL · Node.js/Express · React/TypeScript · Jest + Cypress

---

## 1. Overview

**Goal:** A web page that lets authenticated users view their tasks and create new ones.
**Out of scope:** editing tasks, deleting tasks, filtering (Phase 2).

---

## 2. Functional Requirements (EARS)

### 2.1 Ubiquitous

```
The API SHALL require a valid JWT Bearer token on every endpoint.
The API SHALL return all error responses as JSON — never raw stack traces.
The UI SHALL NOT make any API call before the user is authenticated.
```

### 2.2 Task List (GET /api/tasks)

```
WHEN an authenticated user loads the dashboard,
  the system SHALL fetch GET /api/tasks
  AND SHALL display the tasks in a list, most recent first.

WHEN the task list is loading,
  the system SHALL display a loading spinner.

WHEN the task list is empty,
  the system SHALL display "No tasks yet. Create your first one."

WHEN the API returns an error,
  the system SHALL display "Failed to load tasks. Try again." with a Retry button.
```

### 2.3 Create Task (POST /api/tasks)

```
WHEN an authenticated user submits the create-task form with a valid title,
  the system SHALL send POST /api/tasks
  AND SHALL add the new task to the top of the list without a full page reload.

IF title is blank or missing,
  THEN the system SHALL return HTTP 400 with error VALIDATION_ERROR
  AND SHALL NOT persist the task.

IF title exceeds 255 characters,
  THEN the system SHALL return HTTP 400 with error VALIDATION_ERROR.

WHEN the task is created successfully,
  the system SHALL clear the form and show a "Task created" success message for 3 seconds.
```

### 2.4 Pagination

```
WHEN the task list has more than 20 tasks,
  the system SHALL paginate results (20 per page)
  AND SHALL display "Next" and "Previous" buttons.

The API SHALL accept `page` (default 0) and `size` (default 20, max 100) query params.
```
