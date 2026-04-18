# Project Constitution — Notification Service

> **Version:** 1.0 | **Status:** 🟢 Approved | **Date:** 2026-01-15 | **Author:** Platform Team  
> This document is the non-negotiable rulebook for this project. All AI agents and developers must follow it before writing a single line of code.

---

## 1. Project Overview

The Notification Service is a standalone REST API that sends email and SMS notifications on behalf of other internal services. It accepts a notification request (channel, recipient, template, variables), renders the message, queues it for delivery, and tracks delivery status. It does not own user data — it receives everything it needs in the request payload.

---

## 2. Architectural DNA

| Concern | Decision |
|---------|----------|
| Architecture pattern | Layered: Router → Service → Repository |
| State management | Stateless HTTP; delivery state persisted in PostgreSQL |
| Service boundaries | Notification Service is isolated — no direct DB access from callers |
| Naming conventions | `snake_case` for everything: files, functions, variables, DB columns |
| Folder structure | `routers/` · `services/` · `repositories/` · `schemas/` · `models/` |
| Approved libraries | FastAPI, SQLAlchemy, Alembic, Pydantic v2, pytest, httpx — NO others without approval |

---

## 3. The Gates (Non-Negotiable Rules)

### 🔵 Simplicity Gate
- Reuse before creating — check existing services and utilities first
- Extend before replacing — prefer adding a parameter over a new function
- Ask before adding a new dependency

### 🔴 Anti-Abstraction Gate
- No speculative generalization ("we might need this later")
- No unnecessary design patterns (no factory for a single implementation)
- No unused interfaces or abstract base classes

### 🔒 Security Gate
- No plain-text passwords or tokens — ever
- No stack traces in API responses — use structured error bodies
- No hardcoded credentials in code or config files
- Input validation on ALL entry points via Pydantic schemas
- Recipient PII (email, phone) must never appear in logs
- API key authentication required on all endpoints

---

## 4. Technical Blueprint

### 4.1 Data Model

```
[Notification] (N) ──── (1) [Template]
[Notification] (1) ──── (N) [DeliveryAttempt]
```

| Entity | Key Fields | Notes |
|--------|-----------|-------|
| Notification | id, channel, recipient_ref, template_id, status, created_at | recipient_ref is an opaque ID — no PII stored |
| Template | id, channel, slug, subject_template, body_template, version | slug is unique per channel |
| DeliveryAttempt | id, notification_id, attempted_at, outcome, provider_ref | outcome: sent / failed / bounced |

### 4.2 API Contract Governance

API contracts are defined per feature in `specs/active/[FEAT-XXX]/requirements.md` — not here.
This section defines the rules for how contracts are managed:

- Contracts are written and frozen **before any implementation begins**
- No contract may change without human approval and a spec update first
- A breaking change to a contract requires a version bump and cross-team notification
- The source of truth for any contract is always `requirements.md`, not the code

### 4.3 Security Plan

- All endpoints require `X-API-Key` header; validated in a FastAPI dependency
- API keys stored hashed in environment config — never in code
- Recipient addresses resolved server-side from `recipient_ref` via internal lookup — raw emails/phones never accepted from callers
- Pydantic schemas enforce field types and constraints at the HTTP boundary
- No sensitive fields logged; structlog context vars used for safe correlation IDs only

### 4.4 Rollout Strategy

- Feature flag: No — initial launch only
- DB migration plan: Alembic migration per schema change; `alembic upgrade head` in deployment pipeline
- Rollback plan: `alembic downgrade -1`; previous container image tagged and retained for 7 days

---

## 5. Risk Map

| Decision | Worst Case | How We'll Know | Escape Plan |
|----------|------------|----------------|-------------|
| SQLAlchemy async session | Deadlock under concurrent writes | Load test shows timeouts | Switch to sync sessions; revisit connection pool config |
| Pydantic v2 | Breaking migration from v1 internal libs | Import errors at startup | Pin to v1 in requirements.txt; open migration task |
| External SMTP provider | Delivery failure / rate limit | Delivery attempt outcome = failed; alert fires | Switch provider env var; retry queue drains automatically |

### Library Validation Checklist

Before any new library is added:
- [x] Compatible with Python 3.12?
- [x] Actively maintained (last commit < 6 months)?
- [x] License compatible with project (MIT/Apache)?
- [x] No known critical CVEs?

---

## 6. Do NOT List

- Do NOT modify `repositories/base.py` — it is shared by all repositories
- Do NOT change existing API contracts without approval
- Do NOT introduce new frameworks without approval
- Do NOT refactor code outside the current task scope
- Do NOT skip the human approval gate
- Do NOT store raw recipient addresses in the `notifications` table

---

## 7. Human Approval Gate ✅

- [x] Plan meets all requirements
- [x] Plan obeys the constitution
- [x] API contracts are explicit and complete
- [x] Security is planned, not an afterthought
- [x] Risk map is filled for every major decision
- [x] Rollout / rollback strategy is defined
- [x] Reviewed and signed off by: **Platform Team** on **2026-01-15**

> 🔒 **Intent is now frozen.** The AI agent may begin execution.

---

## 8. Clarification Gate

> All items resolved — see `CLARIFICATION_GATE.md`.

---

## 9. Out of Scope

- User preference management (opt-in / opt-out) — separate service
- Real-time delivery status webhooks — v2
- Multi-language template rendering — v2
- Unsubscribe link generation — v2
