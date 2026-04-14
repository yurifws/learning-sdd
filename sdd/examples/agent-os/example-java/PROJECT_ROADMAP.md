# PROJECT_ROADMAP.md — Task Management API

> This is a filled-in example. See the blank template at `../PROJECT_ROADMAP.md`.

---

## Current Phase

**Phase:** Phase 1 — Core Task API
**Goal:** Deliver authenticated CRUD endpoints for the Task domain with status transition validation.

---

## Phases

| Phase | Name | Status | Goal |
|---|---|---|---|
| 1 | Core Task API | 🔄 In Progress | CRUD + status transitions + JWT auth |
| 2 | Assignment & Users | ⏳ Planned | Assign tasks to users, user management endpoints |
| 3 | Audit Log | ⏳ Planned | Record every status change with actor and timestamp |
| 4 | Notifications | ⏳ Planned | Email on assignment and completion |

> Status options: 🔄 In Progress · ✅ Done · ⏳ Planned · 🚫 Blocked

---

## What's Next (Backlog)

1. `POST /tasks` — create a task
2. `GET /tasks/{id}` — get task by ID
3. `GET /tasks` — list tasks (paginated)
4. `PATCH /tasks/{id}/status` — transition status with validation
5. `DELETE /tasks/{id}` — soft delete
6. `PATCH /tasks/{id}/assign` — assign to a user (Phase 2)

---

## What's Frozen

- Email notifications — deferred to Phase 4
- WebSocket/SSE real-time updates — deferred indefinitely, needs product decision
- File attachments — out of scope, requires S3 integration not yet approved