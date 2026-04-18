# Clarification Gate — Notification Service

> All questions must be resolved **before** the first requirements are written.  
> Status: ✅ **All resolved** — 2026-01-12

---

## Bucket 1: Functional Scope

**Q1: Does the Notification Service own user data (email addresses, phone numbers)?**  
**A:** No. Callers pass an opaque `recipient_ref` (internal user ID). The service resolves the actual address from an internal User Service at send time. Raw addresses are never persisted or logged.

**Q2: Which channels are in scope for v1?**  
**A:** Email and SMS only. Push notifications are out of scope for v1.

**Q3: Who creates and manages templates — callers at runtime, or the platform team at deploy time?**  
**A:** Templates are managed by the platform team via a separate admin workflow (out of scope for FEAT-001). FEAT-001 covers sending a notification using an existing template by slug.

---

## Bucket 2: Data & State

**Q4: Should the service retry failed deliveries automatically?**  
**A:** Yes. Up to 3 attempts with exponential backoff. If all 3 fail, status is set to `failed` and an alert fires. Retry logic is in scope for FEAT-001.

**Q5: How long should notification records be retained?**  
**A:** 90 days. Purge job is out of scope for FEAT-001 but the `created_at` column must be indexed for it.

---

## Bucket 3: Security & Auth

**Q6: How do callers authenticate?**  
**A:** `X-API-Key` header. Keys are provisioned per-service (not per-user). Validation is a FastAPI dependency — not inline logic.

**Q7: What happens when an API key is invalid?**  
**A:** Return `401 Unauthorized` with body `{"error": "invalid_api_key"}`. No additional detail exposed.

---

## Bucket 4: Integration & Contracts

**Q8: Which email provider is used?**  
**A:** SendGrid for v1. Provider is abstracted behind a `NotificationProvider` protocol — swappable via env var without code changes.

**Q9: What does the success response look like? Does the caller need a tracking ID?**  
**A:** Yes. Response includes `notification_id` (UUID) and `status: "queued"`. Caller can poll `/notifications/{id}/status` for delivery updates.

---

## Bucket 5: Failure Modes

**Q10: What if the User Service is unavailable when resolving `recipient_ref`?**  
**A:** Return `503 Service Unavailable` with body `{"error": "recipient_resolution_failed"}`. Do not queue the notification. Log the failure with correlation ID (no PII).

**Q11: What if an invalid template slug is provided?**  
**A:** Return `422 Unprocessable Entity` with body `{"error": "template_not_found", "slug": "[slug]"}`.

---

> ✅ All 11 questions resolved. No open `[NEEDS_CLARIFICATION]` items.  
> Reviewed by: **Platform Team** on **2026-01-12**
