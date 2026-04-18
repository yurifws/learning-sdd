# Drift Taxonomy — The Four Kinds of Architectural Drift

> Not all drift looks the same. Each kind has its own trigger, its own danger, and its own failure mode. Knowing the type determines where in the pipeline to catch it.

**System:** Order Service API | **Spec version:** 1.0.0 | **Reality:** 18 months of undocumented evolution

---

## Overview

| Type | What changes | Who gets hurt | How fast it surfaces |
|---|---|---|---|
| [Structural](#1-structural-drift) | Shape of the data | Any consumer of that contract | Immediately, on first call |
| [Behavioral](#2-behavioral-drift) | What the system does | Client logic that branches on responses | On the specific code path |
| [Security](#3-security-drift) | Trust boundaries | The entire system | Not until exploited |
| [Evolutionary](#4-evolutionary-drift) | Accumulated undocumented decisions | The whole team | Slowly, invisibly |

---

## 1. Structural Drift

**What it is:** The shape of a request or response changes — a field is renamed, removed, added without a schema update, or its type changes.

**How it was created in the Order Service:**

The spec defined the successful order response as:

```json
{
  "orderId": "f3a1c9b2-...",
  "status": "CREATED"
}
```

Six months in, the mobile team needed delivery estimates. A developer added the field directly in code to unblock the release:

```json
{
  "orderId": "f3a1c9b2-...",
  "status": "CREATED",
  "estimatedDelivery": "2025-07-15"
}
```

Three months after that, a backend refactor renamed `orderId` to `order_id` to align with the database column naming convention:

```json
{
  "order_id": "f3a1c9b2-...",
  "status": "CREATED",
  "estimatedDelivery": "2025-07-15"
}
```

**The spec still says `orderId`. The system returns `order_id`.**

---

**What breaks:**

- The web frontend reads `response.orderId` — it now gets `undefined`. Orders silently fail to display.
- Security scanners looking for PII in `orderId` fields are now blind. The data is still there, the scanner just can't find it.
- Any consumer who integrated against the spec is broken on the first call.

---

**The danger of structural drift:** It is the most immediately visible kind, but only after something breaks. Without a gate that compares the live response shape to the spec, there is no warning — just a downstream incident.

---

## 2. Behavioral Drift

**What it is:** The system's response to a given input changes — different status code, different error condition, different side effect — without the spec being updated.

**How it was created in the Order Service:**

The spec defined the out-of-stock error path as:

```
WHEN a user submits an order with an out-of-stock item
THEN the system SHALL return 422 with { "error": "ITEM_OUT_OF_STOCK", "itemId": "<id>" }
```

A refactor replaced the error constants with a shared library. The library used different naming conventions:

```json
HTTP 422
{ "error": "OUT_OF_STOCK", "item": "99" }
```

Two fields changed (`ITEM_OUT_OF_STOCK` → `OUT_OF_STOCK`, `itemId` → `item`). The spec was not updated. The frontend was not notified.

**Separately**, the spec said inventory validation failures return `404 Not Found`. Under load, the inventory service began timing out. Rather than propagate the timeout correctly, an error handler was added that returned `500 Internal Server Error` on timeout.

**The spec says 404. The system returns 500 under real-world load.**

---

**What breaks:**

- Frontend code that branches on `error === "ITEM_OUT_OF_STOCK"` never fires. Users see a generic error instead of "this item is out of stock."
- Monitoring alerts that trigger on `5xx` rates are now catching inventory timeouts — but the ops team assumes it's a service crash, not a timeout.
- Consumer-driven contracts between services fail silently because nobody validated the error contract.

---

**The danger of behavioral drift:** It only surfaces on specific code paths. The happy path passes every test. The broken path waits for a real condition — out of stock, timeout, edge case — to reveal itself in production.

---

## 3. Security Drift

**What it is:** An undocumented surface is added to the system — an endpoint, a parameter, an auth bypass — that exists outside all security controls, scanners, and rate limiters.

**How it was created in the Order Service:**

The mobile team needed a bulk export of order history for an offline mode feature. They were blocked waiting for the API team to spec and review a new endpoint. Under deadline pressure, a developer added:

```http
GET /orders/export?userId=<id>&format=csv
```

This endpoint:
- Was added directly to the controller
- Was not in the OpenAPI spec
- Was not behind the API gateway (added to an internal route)
- Had no rate limiting
- Had no pagination — it returned all records for a given user

No security review. No spec update. No gate.

---

**What breaks:**

- The API gateway applies auth, rate limiting, and request logging to all known routes. This route is unknown. None of those controls apply.
- WAF rules scan traffic against documented endpoints. This endpoint is invisible to the WAF.
- An attacker who discovers `/orders/export` can enumerate any user's full order history with no throttle.
- The vulnerability exists until someone finds it — internally or externally.

---

**The danger of security drift:** It does not surface as a bug. It surfaces as a breach. The system appears to work correctly. The ghost door is open, and only someone who walks through it will know.

> A developer under deadline pressure created an undocumented endpoint. They planned to circle back and document it. They never did. That is how ghost doors are built — one at a time, by reasonable people, under reasonable pressure.

---

## 4. Evolutionary Drift

**What it is:** The slow accumulation of undocumented decisions. No single change is dramatic. Each one seems minor. Over time, the spec becomes a historical artifact — accurate for the system that was, useless for the system that is.

**How it was created in the Order Service:**

Over 18 months, the following were changed without spec updates:

| Change | Spec says | Reality |
|---|---|---|
| Rate limiting | Not specified | 100 req/min per user, 429 on breach |
| Auth token format | Bearer JWT | Bearer JWT or API key (mobile legacy support) |
| Response envelope | Raw object | `{ "data": { ... }, "meta": { "requestId": "..." } }` |
| Pagination | Not specified | Cursor-based, `nextCursor` field |
| Deprecated fields | Not mentioned | `legacyOrderRef` still returned for backward compatibility |
| Error format | `{ "error": "<code>" }` | `{ "error": { "code": "...", "message": "..." } }` |

Each of these changes was made for a good reason. Each was "temporary" or "minor." None were specced. All of them are now load-bearing.

---

**What breaks:**

- A new developer joins and reads the spec. They build their integration against it. None of it works correctly.
- A new service tries to consume the Order API. It takes three weeks of trial and error to discover the actual response shape.
- The team cannot confidently answer: "what does this API actually do?" Nobody knows the full list of undocumented behaviors.
- Institutional knowledge is now held only by the people who made those changes — or nobody at all.

---

**The danger of evolutionary drift:** It is invisible until it is catastrophic. There is no error, no alert, no incident. Just a growing distance between the team's mental model of the system and what the system actually does. One day, that distance causes a decision that breaks something — and nobody can explain why.

---

## How the Types Compound

In isolation, each drift type is manageable. In combination, they create a system that is impossible to reason about:

```
Structural drift  →  consumers break on first call
Behavioral drift  →  consumers break on specific conditions
Security drift    →  ghost surfaces accumulate outside all controls
Evolutionary drift → the spec becomes a lie, institutional knowledge vanishes
```

The Order Service has all four. The spec says one thing. Reality is something else entirely.

That is the system you are actually running.
