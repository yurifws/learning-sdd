# PROJECT_PLAN.md — Task Management API

> This is a filled-in example. See the blank template at `../PROJECT_PLAN.md`.

---

## Mission

A REST API that lets teams create, assign, and track tasks — with JWT-secured endpoints and a clean audit trail of status changes.

---

## What We Are Building

- Task CRUD: create, read, update, delete tasks
- Assignment: assign a task to a user
- Status transitions: TODO → IN_PROGRESS → DONE (with validation)
- Audit log: every status change is recorded with timestamp and actor
- JWT authentication: all endpoints require a valid token
- OpenAPI documentation: full Swagger UI at `/swagger-ui.html`

---

## What We Are NOT Building

- No frontend — API only
- No real-time notifications (WebSocket, SSE) — deferred to Phase 2
- No file attachments — out of scope
- No multi-tenancy — single organization per deployment
- No email integration in v1

---

## Success Criteria

- All endpoints documented in OpenAPI 3 and returning correct HTTP status codes
- Every endpoint covered by at least one integration test using Testcontainers
- Status transition rules enforced at the domain layer (not in the controller)
- JWT validation working end-to-end with a self-signed RS256 key pair
- `./mvnw verify` passes clean on a fresh clone