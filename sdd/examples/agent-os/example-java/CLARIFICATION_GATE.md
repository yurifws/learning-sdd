# CLARIFICATION_GATE.md — Pre-Spec Ambiguity Resolution

> Run this gate BEFORE writing any spec or starting any task.
> Every open question must be resolved before implementation begins.
> Flag anything unclear as `[NEEDS_CLARIFICATION]` and stop — never guess.

---

## Gate Protocol

For every new endpoint or requirement, go through these 5 buckets:

| Bucket | Question type | Examples |
|---|---|---|
| Auth & Roles | Who can call this? What happens without auth? | Which role? 401 vs 403? |
| Data Shape | What does the request/response look like? | Fields, types, nullable? |
| Error Handling | What errors are possible? How are they surfaced? | 404 message, 409 body |
| Edge Cases | What are the boundary states? | Empty list vs null? Status collision? |
| Data Ownership | Can actor A modify data belonging to actor B? | User editing another user's task? |

---

## Session — Phase 1: Core Task API

> Held before writing `requirements.md`. All items resolved before spec was written.

---

### Q1: Who can create, update, and delete tasks?

**Question:** Is task management admin-only, or can any authenticated user manage their own tasks?

**Resolution:** Phase 1 is admin-only. `ROLE_ADMIN` required for POST, PUT, PATCH, DELETE.
Any authenticated user (`ROLE_USER` or higher) can read tasks.

**Why:** Team management use case. Tasks are created by project managers (admins), not self-assigned by users.

**Resolved:** 2025-01-01

---

### Q2: What are the valid task status transitions?

**Question:** Can a task go from DONE back to TODO? Can it skip IN_PROGRESS?

**Resolution:**
- `TODO → IN_PROGRESS` ✅
- `IN_PROGRESS → DONE` ✅
- `IN_PROGRESS → TODO` ✅ (unblock — return to queue)
- `TODO → DONE` ❌ (must go through IN_PROGRESS)
- Any → TODO from DONE ❌ (done is terminal in Phase 1)

Invalid transitions return HTTP 422 with code `INVALID_TRANSITION`.

**Resolved:** 2025-01-01

---

### Q3: What does a "soft delete" mean for tasks?

**Question:** Does DELETE remove the row, or mark it inactive?

**Resolution:** Soft delete — set `status = DELETED`. The row stays in the database.
Deleted tasks do not appear in `GET /tasks` list. They can be retrieved by ID with a dedicated admin endpoint (Phase 2).

**Impact:** `TaskStatus` enum includes `DELETED`. No `DELETE FROM` SQL ever runs.

**Resolved:** 2025-01-01

---

### Q4: What fields are required when creating a task?

**Question:** Is `description` required? Is `assigneeId` required at creation time?

**Resolution:**
- `title` — required (max 255 chars)
- `description` — optional (nullable)
- `assigneeId` — optional at creation (can be assigned later via PATCH)
- `dueDate` — optional (ISO-8601 date, e.g., `2025-03-01`)

**Resolved:** 2025-01-01

---

### Q5: Can a user reassign a task to themselves or to another user?

**Zero-tolerance zone — data ownership.**

**Question:** Can a ROLE_USER assign any task to any user, or only to themselves?

**Resolution:** Phase 1 — assignment is admin-only (`ROLE_ADMIN`). Users cannot self-assign.
Phase 2 will introduce `ROLE_USER` self-assignment with approval workflow.

**Resolved:** 2025-01-01

---

### Q6: What is the paginated list default behavior?

**Question:** Default sort order? Max page size? Filter by status?

**Resolution:**
- Default sort: `createdAt DESC`
- Default page size: 20
- Max page size: 100 (return 400 if exceeded)
- Status filter: optional query param `?status=TODO` (no filter = return all non-DELETED)

**Resolved:** 2025-01-01

---

## Gate Sign-Off

| Item | Status |
|---|---|
| All 5 buckets reviewed | ✅ |
| Role requirements defined for all endpoints | ✅ |
| Status transitions documented | ✅ |
| Soft-delete behavior defined | ✅ |
| Required vs optional fields confirmed | ✅ |
| Data ownership rule confirmed | ✅ |
| Pagination defaults confirmed | ✅ |
| No unresolved `[NEEDS_CLARIFICATION]` items | ✅ |

> Gate passed. Writing `requirements.md` may begin.
