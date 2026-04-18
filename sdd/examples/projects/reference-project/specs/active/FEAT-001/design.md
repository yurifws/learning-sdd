# Design — FEAT-001: Send Notification

> **Requirements reference:** `requirements.md`  
> **Constitution reference:** `CONSTITUTION.md`  
> **Status:** 🔒 Frozen | **Date:** 2026-01-15

---

## 1. Package Structure

```
app/
├── main.py                          # App factory; registers routers + exception handlers
├── dependencies.py                  # verify_api_key(), get_db_session()
├── routers/
│   └── notifications.py             # POST /notifications, GET /notifications/{id}
├── services/
│   └── notification_service.py      # send_notification(), get_notification()
├── repositories/
│   ├── base.py                      # BaseRepository (shared — do not modify)
│   └── notification_repo.py         # NotificationRepository, DeliveryAttemptRepository
├── schemas/
│   └── notification.py              # SendNotificationRequest, NotificationResponse, StatusResponse
├── models/
│   └── notification.py              # Notification, Template, DeliveryAttempt ORM models
└── providers/
    └── sendgrid_provider.py         # SendGrid implementation of NotificationProvider protocol
```

---

## 2. Data Models

### ORM Models (SQLAlchemy)

```python
class Notification(Base):
    __tablename__ = "notifications"
    id: str                  # UUID, primary key, prefix "ntf_"
    channel: str             # "email" | "sms"
    recipient_ref: str       # opaque user ID — never a real address
    template_id: str         # FK → templates.id
    variables: dict          # JSONB
    status: str              # "queued" | "delivered" | "failed"
    created_at: datetime     # indexed

class Template(Base):
    __tablename__ = "templates"
    id: str
    channel: str
    slug: str                # unique per channel
    subject_template: str
    body_template: str
    version: int

class DeliveryAttempt(Base):
    __tablename__ = "delivery_attempts"
    id: str
    notification_id: str     # FK → notifications.id
    attempted_at: datetime
    outcome: str             # "sent" | "failed" | "bounced"
    provider_ref: str        # provider-assigned message ID
```

### Migrations

- `migrations/versions/0001_create_notifications.py` — creates all three tables
- Run with: `alembic upgrade head`

---

## 3. API Endpoints

### POST /notifications

```
Auth:    X-API-Key header (verified by verify_api_key dependency)
Input:   SendNotificationRequest (Pydantic)
Output:  202 Accepted → NotificationResponse
Errors:  401 (bad key) · 422 (validation) · 422 (bad template) · 503 (user svc down)
```

**Flow:**
1. `verify_api_key` dependency validates header
2. Pydantic validates request body
3. `notification_service.send_notification()`:
   a. Look up template by `channel` + `slug` → 422 if not found
   b. Resolve recipient from User Service via `recipient_ref` → 503 if unavailable
   c. Create `Notification` record with status `"queued"`
   d. Enqueue delivery task (background task)
4. Return `202` with `notification_id` + `status: "queued"`

### GET /notifications/{id}

```
Auth:    X-API-Key header
Input:   Path param: notification_id
Output:  200 OK → StatusResponse (status + delivery_attempts)
Errors:  401 (bad key) · 404 (unknown id)
```

---

## 4. Pydantic Schemas

```python
class SendNotificationRequest(BaseModel):
    channel: Literal["email", "sms"]
    recipient_ref: str          # min_length=1
    template_slug: str          # min_length=1
    variables: dict[str, str] = {}

class NotificationResponse(BaseModel):
    notification_id: str
    status: str

class DeliveryAttemptResponse(BaseModel):
    attempted_at: datetime
    outcome: str
    provider_ref: str

class StatusResponse(BaseModel):
    notification_id: str
    channel: str
    template_slug: str
    status: str
    created_at: datetime
    delivery_attempts: list[DeliveryAttemptResponse]
```

---

## 5. Exception Handling

All exceptions are caught by a global handler registered in `main.py`. Services raise domain exceptions; the handler maps them to HTTP responses:

| Exception class | HTTP status | Response body |
|----------------|-------------|---------------|
| `InvalidApiKeyError` | 401 | `{"error": "invalid_api_key"}` |
| `TemplateNotFoundError` | 422 | `{"error": "template_not_found", "slug": "..."}` |
| `RecipientResolutionError` | 503 | `{"error": "recipient_resolution_failed"}` |
| `NotificationNotFoundError` | 404 | `{"error": "notification_not_found"}` |
| Pydantic `ValidationError` | 422 | `{"error": "validation_error", "detail": [...]}` |
| Unhandled `Exception` | 500 | `{"error": "internal_error"}` — no stack trace |

---

## 6. Provider Abstraction

```python
class NotificationProvider(Protocol):
    async def send(self, channel: str, address: str, subject: str, body: str) -> str:
        ...  # returns provider_ref
```

`SendGridProvider` implements this protocol. Provider is injected into `NotificationService` via constructor — swappable without modifying service code.

---

## 7. Retry Logic

Implemented as a background task (FastAPI `BackgroundTasks`):

```
Attempt 1 → wait 30s → Attempt 2 → wait 120s → Attempt 3 → wait 480s → FAILED
```

Each attempt creates a `DeliveryAttempt` row regardless of outcome. After 3 failures, `Notification.status` is set to `"failed"`.
