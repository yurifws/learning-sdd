# Requirements — FEAT-001: Send Notification

> **Spec reference:** `CONSTITUTION.md` · `CLARIFICATION_GATE.md`  
> **Status:** 🔒 Frozen | **Date:** 2026-01-15 | **Author:** Platform Team

---

## 1. Feature Summary

A caller can send a notification (email or SMS) by providing a channel, a recipient reference, a template slug, and template variables. The service validates the request, resolves the recipient, renders the message, and queues it for delivery. Delivery is attempted asynchronously with up to 3 retries.

---

## 2. EARS Requirements

### 2.1 Happy Path

**EARS-01** (Ubiquitous)  
The system SHALL accept `POST /notifications` with a valid API key, valid request body, resolvable `recipient_ref`, and existing template slug, and SHALL return `202 Accepted` with a `notification_id` and `status: "queued"`.

**EARS-02** (Event-Driven)  
When a notification is queued, the system SHALL attempt delivery via the configured provider within 5 seconds.

**EARS-03** (Event-Driven)  
When delivery succeeds, the system SHALL update the notification status to `"delivered"` and record a `DeliveryAttempt` with `outcome: "sent"`.

**EARS-04** (Ubiquitous)  
The system SHALL accept `GET /notifications/{id}` with a valid API key and SHALL return the notification's current `status` and the list of `delivery_attempts`.

---

### 2.2 Retry Behavior

**EARS-05** (Event-Driven)  
When a delivery attempt fails, the system SHALL schedule a retry with exponential backoff: 30 s, 2 min, 8 min.

**EARS-06** (State-Driven)  
While a notification has fewer than 3 failed delivery attempts, the system SHALL continue retrying.

**EARS-07** (Event-Driven)  
When a notification reaches 3 failed delivery attempts, the system SHALL set status to `"failed"` and SHALL NOT retry further.

---

### 2.3 Validation & Error Handling

**EARS-08** (Unwanted Behavior)  
If the `X-API-Key` header is missing or invalid, the system SHALL return `401 Unauthorized` with body `{"error": "invalid_api_key"}` and SHALL NOT process the request.

**EARS-09** (Unwanted Behavior)  
If the request body is malformed or missing required fields, the system SHALL return `422 Unprocessable Entity` with a structured error body listing the invalid fields.

**EARS-10** (Unwanted Behavior)  
If the template slug does not exist for the given channel, the system SHALL return `422 Unprocessable Entity` with body `{"error": "template_not_found", "slug": "[slug]"}`.

**EARS-11** (Unwanted Behavior)  
If the User Service is unavailable during `recipient_ref` resolution, the system SHALL return `503 Service Unavailable` with body `{"error": "recipient_resolution_failed"}` and SHALL NOT queue the notification.

**EARS-12** (Unwanted Behavior)  
If `GET /notifications/{id}` is called with a valid API key but an unknown `id`, the system SHALL return `404 Not Found` with body `{"error": "notification_not_found"}`.

---

## 3. Request / Response Contract

### POST /notifications

**Request body:**
```json
{
  "channel": "email",
  "recipient_ref": "usr_abc123",
  "template_slug": "welcome_email",
  "variables": {
    "first_name": "Alex"
  }
}
```

**Success response — 202 Accepted:**
```json
{
  "notification_id": "ntf_7f3a9b",
  "status": "queued"
}
```

### GET /notifications/{id}

**Success response — 200 OK:**
```json
{
  "notification_id": "ntf_7f3a9b",
  "channel": "email",
  "template_slug": "welcome_email",
  "status": "delivered",
  "created_at": "2026-01-15T10:00:00Z",
  "delivery_attempts": [
    {
      "attempted_at": "2026-01-15T10:00:05Z",
      "outcome": "sent",
      "provider_ref": "sg_msg_xyz"
    }
  ]
}
```

---

## 4. Success Criteria (Acceptance Criteria)

| # | When... | The system SHALL... | Verified by |
|---|---------|---------------------|-------------|
| AC-01 | Valid POST with all fields | Return 202 with notification_id | `pytest tests/test_notifications.py::test_send_notification_returns_202` |
| AC-02 | Valid POST | Create a Notification row with status "queued" | `pytest tests/test_notifications.py::test_send_notification_persists_record` |
| AC-03 | Delivery succeeds | Update status to "delivered" | `pytest tests/test_notifications.py::test_delivery_success_updates_status` |
| AC-04 | GET /notifications/{id} with valid id | Return status + delivery attempts | `pytest tests/test_notifications.py::test_get_notification_returns_status` |

---

## 5. Security / Negative Criteria

| # | To prevent... | The system SHALL... | Verified by |
|---|--------------|---------------------|-------------|
| SEC-01 | Unauthenticated access | Reject missing/invalid API key with 401 | `pytest tests/test_notifications.py::test_missing_api_key_returns_401` |
| SEC-02 | PII leakage in logs | Never log recipient address | `pytest tests/test_notifications.py::test_no_pii_in_log_output` |
| SEC-03 | Stack trace exposure | Return structured error body, not exception detail | `pytest tests/test_notifications.py::test_error_response_has_no_traceback` |
