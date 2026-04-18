# tasks.md — Notification Service / FEAT-001: Send Notification

> **Spec reference:** `requirements.md` and `design.md`  
> **Constitution:** `CONSTITUTION.md`  
> **Status:** 🔴 In Progress | **Date:** 2026-01-16
>
> AI agent: work through this file **top to bottom, one task at a time.**
> Check off each task only when the verify command passes.

---

## Ground Rules (Read Before Starting)

- Touch **only the files listed** in each task — nothing else
- Each task has **one objective** — no side-quests, no unrequested refactoring
- If a task feels too big or ambiguous, **stop and ask** before proceeding
- `[p]` = safe to parallelize — only if the file boundaries don't overlap
- Mark a task `✅` only when the verify step passes
- If a verify command fails for a reason unrelated to your task, **flag it** — do not silently fix it

---

## Phase 1 — Project Setup

- [ ] **T-01** · Initialize project structure  
  **Files:** `requirements.txt`, `app/main.py`, `app/dependencies.py`  
  **Objective:** Create the FastAPI app factory, register an empty router, and define the `verify_api_key` dependency stub  
  **Verify:** `uvicorn app.main:app --reload` starts without errors; `GET /` returns 404  
  **Ref:** Constitution § Tech Stack

- [ ] **T-02** · Configure environment variables  
  **Files:** `.env.example`, `app/config.py`  
  **Objective:** Define all required env vars (`API_KEY_HASH`, `DATABASE_URL`, `USER_SERVICE_URL`, `SENDGRID_API_KEY`) and a Pydantic settings class  
  **Verify:** Application starts without missing configuration errors when `.env.example` values are loaded  
  **Ref:** Constitution § Security Gate

---

## Phase 2 — Data Foundation

- [ ] **T-03** · Create DB schema migrations  
  **Files:** `migrations/versions/0001_create_notifications.py`  
  **Objective:** Define all three tables (`notifications`, `templates`, `delivery_attempts`) with correct columns, constraints, and indexes  
  **Verify:** `alembic upgrade head` runs cleanly; schema matches `design.md` § Data Models  
  **Ref:** `design.md` § Data Models

- [ ] **T-04 [p]** · Create ORM models  
  **Files:** `app/models/notification.py`  
  **Objective:** Define `Notification`, `Template`, and `DeliveryAttempt` SQLAlchemy models matching the migration  
  **Verify:** `python -c "from app.models.notification import Notification, Template, DeliveryAttempt"` — no import errors  
  **Ref:** `design.md` § Data Models

- [ ] **T-05 [p]** · Create repository layer  
  **Files:** `app/repositories/notification_repo.py`  
  **Objective:** Implement `NotificationRepository` and `DeliveryAttemptRepository` with `create`, `get_by_id`, `update_status`, and `get_attempts` methods  
  **Verify:** `pytest tests/test_notification_repo.py` passes  
  **Ref:** `design.md` § Package Structure

---

## Phase 3 — API Contracts

- [ ] **T-06** · Define Pydantic schemas  
  **Files:** `app/schemas/notification.py`  
  **Objective:** Implement `SendNotificationRequest`, `NotificationResponse`, `DeliveryAttemptResponse`, and `StatusResponse` matching `design.md` § Pydantic Schemas  
  **Verify:** `mypy app/schemas/` — no type errors; `python -c "from app.schemas.notification import SendNotificationRequest"` — no errors  
  **Ref:** `design.md` § Pydantic Schemas

- [ ] **T-07** · Create notification router with stubs  
  **Files:** `app/routers/notifications.py`, `app/main.py`  
  **Objective:** Register `POST /notifications` and `GET /notifications/{id}` routes returning stub `501 Not Implemented` responses  
  **Verify:** `curl -H "X-API-Key: test" http://localhost:8000/notifications` returns `501`  
  **Ref:** `design.md` § API Endpoints

---

## Phase 4 — Core Logic

- [ ] **T-08** · Implement provider abstraction  
  **Files:** `app/providers/sendgrid_provider.py`  
  **Objective:** Define `NotificationProvider` protocol and implement `SendGridProvider.send()` using the SendGrid HTTP API  
  **Verify:** `pytest tests/test_sendgrid_provider.py` passes (provider tested with mocked HTTP)  
  **Ref:** `design.md` § Provider Abstraction

- [ ] **T-09** · Implement `send_notification` — happy path  
  **Files:** `app/services/notification_service.py`  
  **Objective:** Implement the full send flow: template lookup → recipient resolution → create notification record → enqueue background delivery  
  **Verify:** `pytest tests/test_notifications.py::test_send_notification_returns_202 tests/test_notifications.py::test_send_notification_persists_record` pass  
  **Ref:** `requirements.md` — EARS-01; `design.md` § POST /notifications

- [ ] **T-10** · Implement delivery background task and retry logic  
  **Files:** `app/services/notification_service.py`  
  **Objective:** Implement the background delivery task with exponential backoff retry (3 attempts), DeliveryAttempt creation, and status updates  
  **Verify:** `pytest tests/test_notifications.py::test_delivery_success_updates_status tests/test_notifications.py::test_retry_scheduled_after_failure tests/test_notifications.py::test_notification_fails_after_max_retries` pass  
  **Ref:** `requirements.md` — EARS-02, EARS-03, EARS-05, EARS-06, EARS-07

- [ ] **T-11** · Implement `get_notification`  
  **Files:** `app/services/notification_service.py`  
  **Objective:** Implement lookup by id; raise `NotificationNotFoundError` if not found  
  **Verify:** `pytest tests/test_notifications.py::test_get_notification_returns_status tests/test_notifications.py::test_unknown_notification_id_returns_404` pass  
  **Ref:** `requirements.md` — EARS-04, EARS-12

- [ ] **T-12** · Wire service into router  
  **Files:** `app/routers/notifications.py`  
  **Objective:** Replace stubs with real service calls; map domain exceptions to correct HTTP responses  
  **Verify:** `pytest tests/test_notifications.py` — all 11 functional tests pass  
  **Ref:** `design.md` § Exception Handling

---

## Phase 5 — Security

- [ ] **T-13** · Implement API key authentication  
  **Files:** `app/dependencies.py`  
  **Objective:** Implement `verify_api_key` to compare hashed incoming key against `API_KEY_HASH` env var; raise `InvalidApiKeyError` if invalid  
  **Verify:** `pytest tests/test_notifications.py::test_missing_api_key_returns_401` passes  
  **Ref:** `requirements.md` — EARS-08; Constitution § Security Gate

- [ ] **T-14** · Register global exception handler  
  **Files:** `app/main.py`  
  **Objective:** Register a FastAPI exception handler that maps domain exceptions to structured HTTP responses with no stack traces  
  **Verify:** `pytest tests/test_notifications.py::test_error_response_has_no_traceback` passes  
  **Ref:** `design.md` § Exception Handling; Constitution § Security Gate

- [ ] **T-15** · Confirm no PII in logs  
  **Files:** `app/services/notification_service.py`, `app/routers/notifications.py`  
  **Objective:** Audit all log statements; ensure no resolved email/phone addresses appear; use correlation IDs only  
  **Verify:** `pytest tests/test_notifications.py::test_no_pii_in_log_output` passes  
  **Ref:** Constitution § Security Gate

---

## Phase 6 — Tests & Verification

- [ ] **T-16 [p]** · Write integration tests for error cases  
  **Files:** `tests/test_notifications.py`  
  **Objective:** Confirm all error path tests (T-07 through T-11, SEC tests) are passing end-to-end  
  **Verify:** `pytest tests/test_notifications.py -v` — all 13 tests pass  
  **Ref:** `requirements.md` § Error Handling

- [ ] **T-17** · Full suite + Definition of Done  
  **Files:** (read-only audit)  
  **Objective:** Run full test suite, verify no regressions, confirm `requirements.md` / `design.md` / `tasks.md` reflect final implementation  
  **Verify:** `pytest` — all tests pass; `mypy app/` — no errors; `ruff check .` — no violations  
  **Ref:** `GUIDED_EXECUTION.md` § Definition of Done

---

## Human Review Checklist

- [x] Every EARS requirement from `requirements.md` maps to at least one task
- [x] No task touches more than 1–2 files
- [x] No task has more than one objective
- [x] Every task has a specific, runnable verify command
- [x] Tasks are ordered correctly — no task depends on one listed after it
- [x] `[p]` tasks have no overlapping files or shared state
- [x] No mega-tasks hiding as a single line
- [x] Reviewed by: **Platform Team** on **2026-01-16**

> **Approved. Agent may begin at T-01.**
