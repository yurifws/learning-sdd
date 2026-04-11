# Requirements — Expired Link Handling

> **Feature ID:** FEAT-042
> **Date:** 2026-04-11
> **Author:** Platform Team
> **Status:** Approved

---

## 1. Overview

**Goal:** When a user clicks a short link that has passed its expiry date, the system must clearly communicate the expiry and never redirect the user to the destination URL.

**Stakeholder:** Growth & Platform Team  
**Priority:** High — expired links currently redirect to destination silently (data integrity bug)

---

## 2. Functional Requirements (EARS Notation)

### 2.1 Ubiquitous (Always true)

```
The system SHALL record a click_event for every request to a short link,
  regardless of whether the link is active, expired, or non-existent.

The system SHALL respond to all link resolution requests within 200ms at p99.

The system SHALL never expose the destination URL in any error response.
```

---

### 2.2 Event-Driven — Expired link accessed

```
WHEN the user sends a GET request to /r/{token},
  IF the link exists AND its expires_at timestamp is in the past,
  THEN the system SHALL return HTTP 410 (Gone)
  AND SHALL render the expired-link error page
  AND SHALL NOT redirect the user to the destination URL.
```

---

### 2.3 Event-Driven — Non-existent link accessed

```
WHEN the user sends a GET request to /r/{token},
  IF no link record exists for the given token,
  THEN the system SHALL return HTTP 404 (Not Found)
  AND SHALL render the not-found error page.
```

---

### 2.4 Event-Driven — Active link accessed

```
WHEN the user sends a GET request to /r/{token},
  IF the link exists AND expires_at is null or in the future,
  THEN the system SHALL issue HTTP 302 to the destination URL
  AND SHALL increment the link's click_count by 1.
```

---

### 2.5 State-Driven — Link with no expiry set

```
WHILE a link's expires_at field is null,
  the system SHALL treat the link as permanently active
  and SHALL never return HTTP 410 for it.
```

---

### 2.6 Unwanted Behavior (Forbidden Paths)

```
IF the link is expired,
  THEN the system SHALL NOT return HTTP 302.
  THEN the system SHALL NOT include the destination URL in any response body or header.
  THEN the system SHALL NOT update click_count for the expired access.

IF the token contains characters outside [a-zA-Z0-9_-],
  THEN the system SHALL return HTTP 400
  AND SHALL NOT attempt any database lookup.
```

---

## 3. Non-Functional Requirements

| Category | Requirement |
|---|---|
| Performance | Link resolution p99 < 200ms under 1,000 req/s |
| Security | Destination URL never leaked in 4xx/5xx responses |
| Observability | Every click_event logged with: token, status (ACTIVE/EXPIRED/NOT_FOUND), timestamp, user_agent |
| Resilience | If click_event write fails, link resolution must still complete (best-effort logging) |

---

## 4. Out of Scope

- Link owner notification when their link expires (separate feature)
- Admin UI to manually un-expire a link
- Custom expiry pages per link owner
- Bulk expiry management

---

## 5. Open Questions

| # | Question | Owner | Due |
|---|---|---|---|
| 1 | Should the 410 page show the original creation date of the link? | @product | 2026-04-14 |

---

## 6. Acceptance Criteria

- [ ] GET /r/{token} with expired link returns HTTP 410 and no redirect
- [ ] GET /r/{token} with active link returns HTTP 302 to destination URL
- [ ] GET /r/{token} with unknown token returns HTTP 404
- [ ] GET /r/{token} with token containing invalid characters returns HTTP 400 — no DB hit
- [ ] Expired link response body does NOT contain the destination URL
- [ ] click_event is recorded for every request (active, expired, not-found)
- [ ] click_count is NOT incremented for expired or not-found requests
- [ ] Links with null expires_at are always treated as active
- [ ] p99 response time < 200ms under load test (1,000 req/s, 60s)
- [ ] All acceptance criteria covered by integration tests using Testcontainers
