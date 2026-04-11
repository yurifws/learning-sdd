# Design — Expired Link Handling

> **Feature ID:** FEAT-042
> **Linked:** requirements.md
> **Date:** 2026-04-11

---

## 1. Architecture Overview

```
GET /r/{token}
      │
      ▼
LinkResolutionController      ← validates token format, delegates
      │
      ▼
LinkResolutionService         ← resolves link state (ACTIVE / EXPIRED / NOT_FOUND)
      │                       ← records click_event (best-effort, async)
      ▼
LinkRepository                ← single DB lookup by token
      │
      ▼
PostgreSQL (short_links + click_events tables)
```

No new layers. The expired-link path is a new branch inside the existing resolution flow.

---

## 2. Link State Model

The resolution logic produces one of three states:

```
ACTIVE     → link exists, expires_at is null or in the future  → HTTP 302
EXPIRED    → link exists, expires_at is in the past            → HTTP 410
NOT_FOUND  → no record for this token                          → HTTP 404
```

This state is an internal enum — never exposed in the response body.

---

## 3. Data Model Changes

### Modified table: `short_links`

Add one column:

| Column | Type | Constraints |
|---|---|---|
| `expires_at` | TIMESTAMP WITH TIME ZONE | nullable |

Index: `idx_short_links_expires_at` on `(expires_at)` — supports future bulk-expiry queries.

Migration: `V5__Add_expiry_to_short_links.sql`

### Existing table: `click_events`

Add one column to record resolution outcome:

| Column | Type | Constraints |
|---|---|---|
| `resolution_status` | VARCHAR(20) | NOT NULL — values: ACTIVE, EXPIRED, NOT_FOUND |

Migration: `V6__Add_resolution_status_to_click_events.sql`

---

## 4. DTO Changes

No new DTOs. Error responses reuse the existing `ErrorResponse` envelope.

### Error responses (existing shape)

**HTTP 410 — Expired**
```json
{
  "error": "LINK_EXPIRED",
  "message": "This link has expired and is no longer available."
}
```

**HTTP 404 — Not Found**
```json
{
  "error": "LINK_NOT_FOUND",
  "message": "This link does not exist."
}
```

**HTTP 400 — Invalid token**
```json
{
  "error": "INVALID_TOKEN",
  "message": "The token contains invalid characters."
}
```

Note: none of these responses include the destination URL — requirement 2.6.

---

## 5. Token Validation

Validated before any DB access (requirement 2.6 — invalid tokens must not hit the DB).

```
Allowed: [a-zA-Z0-9_-]
Max length: 32 characters
Validated by: @Pattern annotation on the controller path variable
```

---

## 6. Click Event Recording

- Written **asynchronously** after the resolution decision is made
- Uses `@Async` + a dedicated thread pool (`clickEventExecutor`)
- If the write fails (DB down, timeout), the failure is logged at WARN and swallowed
- The main resolution response is never blocked by click event persistence

This satisfies the resilience NFR: "if click_event write fails, link resolution must still complete."

---

## 7. Sequence Diagram

```
Client        Controller        Service           Repository       ClickEventAsync
  │                │                │                  │                  │
  │─GET /r/{token}►│                │                  │                  │
  │                │──validate ─────►│                  │                  │
  │                │  token format  │                  │                  │
  │                │                │──findByToken()──►│                  │
  │                │                │◄─Optional<Link>──│                  │
  │                │                │                  │                  │
  │                │          [resolve state]           │                  │
  │                │                │                  │                  │
  │                │                │──────────────────────────────────►  │
  │                │                │   recordClickEvent(token, status)    │
  │                │                │                  │                  │
  │         [ACTIVE]│◄─302 Location──│                  │                  │
  │        [EXPIRED]│◄─410 JSON──────│                  │                  │
  │      [NOT_FOUND]│◄─404 JSON──────│                  │                  │
  │◄───────response─│                │                  │                  │
```

---

## 8. Exception Handling

| Condition | Exception | HTTP | Error Code |
|---|---|---|---|
| Link is expired | `LinkExpiredException` | 410 | LINK_EXPIRED |
| Link not found | `ResourceNotFoundException` | 404 | LINK_NOT_FOUND |
| Invalid token format | (Bean Validation) | 400 | INVALID_TOKEN |

Handled by the existing `GlobalExceptionHandler`. Add two new `@ExceptionHandler` methods.

---

## 9. Technical Decisions

| Decision | Choice | Reason |
|---|---|---|
| Expiry check location | Service layer | Not the DB — clock skew and timezone handling belong in application code |
| Click event persistence | Async / best-effort | Resolution latency must not be coupled to event write latency (NFR: p99 < 200ms) |
| Destination URL in errors | Never included | Explicit requirement 2.6 — security boundary |
| Null `expires_at` semantics | Permanently active | Simpler than a sentinel date; explicit in requirements 2.5 |
