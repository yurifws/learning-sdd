# AGENTS.md — AI Onboarding Packet
# Notification Service

> This file is written FOR the AI agent, not for humans.
> Read this entirely before writing or modifying any code.
> If anything in this file conflicts with another file, this file wins — flag the discrepancy.

---

## 1. Project Overview

**Project:** Notification Service — a REST API that sends email and SMS notifications on behalf of internal services. Callers send a channel, a recipient reference, a template slug, and template variables. The service renders the message, queues it for delivery, and persists delivery status. It does not own user data; recipients are resolved internally from opaque IDs.

**Architecture:** Layered — Router → Service → Repository

**Language & Runtime:** Python 3.12

**Framework:** FastAPI 0.110

---

## 2. Commands

| Task | Command |
|---|---|
| Install dependencies | `pip install -r requirements.txt` |
| Start dev server | `uvicorn app.main:app --reload` |
| Run all tests | `pytest` |
| Run single test | `pytest tests/test_notifications.py` |
| Lint | `ruff check .` |
| Format code | `ruff format .` |
| Type check | `mypy app/` |
| Build / package | `docker build -t notification-service .` |
| Run DB migrations | `alembic upgrade head` |
| Rollback migration | `alembic downgrade -1` |

> Always use the project's virtual environment. Never use globally installed packages.

---

## 3. Project Map

```
notification-service/
├── app/
│   ├── main.py                  # FastAPI app factory, middleware, router registration
│   ├── dependencies.py          # Shared FastAPI dependencies (auth, DB session)
│   ├── routers/
│   │   └── notifications.py     # HTTP route handlers — thin, no business logic
│   ├── services/
│   │   └── notification_service.py  # All business logic lives here
│   ├── repositories/
│   │   ├── base.py              # Shared base — DO NOT MODIFY
│   │   └── notification_repo.py # DB read/write for Notification + DeliveryAttempt
│   ├── schemas/                 # Pydantic request/response models
│   │   └── notification.py
│   └── models/                  # SQLAlchemy ORM models
│       └── notification.py
├── tests/                       # Mirror app/ structure
│   ├── test_notifications.py
│   └── conftest.py              # Fixtures: test DB, test client, mock provider
├── migrations/                  # Alembic migration files
├── .env.example                 # Environment variable template — never touch .env
├── requirements.txt
├── CONSTITUTION.md
├── AGENTS.md                    # This file
└── specs/
    └── active/
        └── FEAT-001/            # Send Notification feature
            ├── requirements.md
            ├── design.md
            ├── scenarios.md
            ├── tasks.md
            └── TEST_FIRST_GATE.md
```

---

## 4. Stack & Conventions

**Naming conventions:**

| Thing | Convention | Example |
|---|---|---|
| Files | snake_case | `notification_service.py` |
| Functions / methods | snake_case | `send_notification` |
| Classes | PascalCase | `NotificationService` |
| DB columns | snake_case | `created_at`, `recipient_ref` |
| Constants | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| Pydantic schemas | PascalCase + `Request`/`Response` suffix | `SendNotificationRequest` |

**Code style rules:**

- Max function length: 25 lines — extract helpers if longer
- No `None` returns from public service methods — raise explicit exceptions
- No commented-out code — delete it or open a task
- No logging of PII — never log `recipient_ref` resolution results, email addresses, or phone numbers
- All DB access goes through the repository layer — no raw queries in services
- Pydantic schemas validate at the HTTP boundary — services receive validated domain objects

**Approved libraries / dependencies:**

| Allowed | Forbidden |
|---|---|
| FastAPI — web framework | Django, Flask — not approved |
| SQLAlchemy 2.x — ORM | Raw psycopg2 queries in services |
| Alembic — DB migrations | Manual `CREATE TABLE` statements |
| Pydantic v2 — validation | Marshmallow, attrs |
| pytest + httpx — testing | unittest.mock for DB — use real test DB |
| structlog — logging | print() statements in production code |
| Any new dependency | Requires human approval before adding |

---

## 5. Git Workflow

- Branch naming: `feat/short-description`, `fix/short-description`, `chore/short-description`
- Commit format: `feat: add send notification endpoint`, `fix: correct retry logic`
- Spec commits first: `spec: update intent for [feature]`, then `plan: ...`, then `feat: ...`
- Always branch from `main`
- Never force-push to `main` or any shared branch
- PRs must pass all tests and checks before merging

---

## 6. Boundaries — Safety Rules

### Always do
- Re-read `CONSTITUTION.md` before starting each task — follow every rule without exception
- Follow spec-first order: update requirements → update design/tasks → implement → verify
- Run `pytest` before marking any task done
- Add or update tests when changing any business logic
- Follow the existing code style of the file you're editing

### Ask before doing
- Adding any new dependency to `requirements.txt`
- Changing the database schema or migrations
- Modifying environment variable names or structure
- Changing a public API contract
- Refactoring more than the files listed in the current task

### Never do
- Read, log, commit, or expose secrets, passwords, API keys, or credentials
- Bypass or weaken the `X-API-Key` authentication dependency
- Log PII — recipient email, phone, or resolved address must never appear in log output
- Delete files without explicit instruction
- Push directly to `main`
- Skip a failing test to make a build pass

---

## 7. Testing Expectations

- Every new feature must have at least one test covering the happy path
- Every EARS clause maps to at least one test — see `specs/active/FEAT-001/TEST_FIRST_GATE.md`
- Write tests before implementation — all tests in `TEST_FIRST_GATE.md` must be failing red before Phase 1 begins
- Test names: `test_[method]_given_[condition]_[expected_result]`
- Test files live in `tests/` mirroring `app/` structure
- No real external calls in unit tests — mock SMTP/SMS provider at system boundary
- Use the real test PostgreSQL database — do not mock the repository layer

---

## 8. Architecture Rules

- Business logic lives in the service layer only — routers are thin (validate, call service, return response)
- Dependencies flow downward: Router → Service → Repository → DB
- Routers never import repositories directly
- All Pydantic validation happens before the service is called
- Exceptions raised in the service layer are caught by a global exception handler in `main.py` — never catch-and-swallow in routers

For the full rule set, see `CONSTITUTION.md`.

---

## Notes for the AI

- When in doubt, ask. A short clarifying question is always better than a wrong assumption.
- If a task touches authentication, authorization, or recipient data — stop and confirm before proceeding.
- Prefer small, focused changes. Avoid large refactors unless explicitly asked.
- This file is the source of truth for operational context. `CONSTITUTION.md` is the source of truth for rules. If they conflict, flag it.
