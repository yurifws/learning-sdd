# runtime-gate.md — Runtime Gate

> Continuous drift detection. Asks one question on repeat:
> Is what is happening in production matching what the spec says should happen?

**Feature:** `POST /orders` — Place a new order

---

## The Rule

- This gate is not a one-time checkpoint — it runs continuously against the live application
- Automated flows follow the exact user paths defined in the spec
- Evidence is captured: logs, API responses, timing
- Evidence is compared to acceptance criteria from the spec

**Critical rule:** If a mismatch is found, you are forbidden from patching the code first.
You must update the spec first. The spec is the system of record. Code follows the spec — never the other way around.

---

## Automated Flow

The runtime gate replays each spec scenario and captures evidence.

```
For each scenario in spec-gate.md:
  1. Execute the exact HTTP request defined in the scenario
  2. Capture: status code, response body, latency, logs
  3. Compare captured evidence to the acceptance criterion
  4. If mismatch → DRIFT DETECTED → open a spec update before any code change
```

---

## Evidence Log

**Run date:** 2025-06-10 | **Environment:** staging | **Build:** `v1.4.2`

---

### ORD-01 — Place valid order → 201

**Request:**
```http
POST /orders
Authorization: Bearer <valid-token>
Idempotency-Key: test-001
Content-Type: application/json

{ "items": [{ "itemId": 42, "quantity": 1 }] }
```

**Expected (from spec):** `201` with `{ "orderId": "<uuid>" }`

**Observed:**
```json
HTTP 201
{ "orderId": "f3a1c9b2-...", "createdAt": "2025-06-10T14:22:01Z" }
```

**Result:** PASS

---

### ORD-05 — Duplicate idempotency key → 200

**Request:** same as above with `Idempotency-Key: test-001`

**Expected (from spec):** `200` with the original `orderId`

**Observed:**
```json
HTTP 200
{ "orderId": "f3a1c9b2-..." }
```

**Result:** PASS

---

### ORD-02 — Out-of-stock item → 422

**Request:** `itemId: 99` (confirmed out of stock in staging)

**Expected (from spec):** `422` with `{ "error": "ITEM_OUT_OF_STOCK", "itemId": "99" }`

**Observed:**
```json
HTTP 422
{ "error": "OUT_OF_STOCK", "item": "99" }
```

**Result:** DRIFT DETECTED

---

## Drift Report

**Requirement:** ORD-02
**Expected error key:** `ITEM_OUT_OF_STOCK`
**Observed error key:** `OUT_OF_STOCK`

**Expected field name:** `itemId`
**Observed field name:** `item`

---

## Resolution — Spec First

> Do NOT patch the implementation. Update the spec first.

**Decision:** The observed response key `OUT_OF_STOCK` and field `item` were agreed on with the frontend team after the spec was written. The spec is out of date.

**Spec update (spec-gate.md, ORD-02):**

Before:
```
THEN the system SHALL return `422` with `{ "error": "ITEM_OUT_OF_STOCK", "itemId": "<id>" }`
```

After:
```
THEN the system SHALL return `422` with `{ "error": "OUT_OF_STOCK", "item": "<id>" }`
```

**After spec is updated and reviewed:** implementation may be adjusted to match — or confirmed already correct.

---

## Runtime Gate Checklist

- [x] Automated flows cover every scenario in the spec
- [x] Evidence is captured for each run (status, body, latency)
- [x] Evidence is compared to acceptance criteria — not to intuition
- [x] Any mismatch triggers a spec update before a code change
- [x] The spec remains the system of record after every run

**Gate status: DRIFT DETECTED on ORD-02 — spec updated, implementation review pending.**
