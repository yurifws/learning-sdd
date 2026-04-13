# spec-gate.md — Spec Gate

> Locks down the what and the why.
> No planning. No architecture. No code. Until this gate passes.

**Feature:** `POST /orders` — Place a new order

---

## The Rule

A requirement only passes this gate when:
- It is testable — specific inputs and outputs, no vague language
- All assumptions are written down
- All `[NEEDS_CLARIFICATION]` tags are resolved
- The Do-Not list is written and reviewed

---

## Requirements

| ID | Requirement |
|---|---|
| ORD-01 | WHEN an authenticated user submits a valid order THEN the system SHALL create the order and return `201` with the order ID |
| ORD-02 | WHEN an order is submitted with an out-of-stock item THEN the system SHALL return `422` with `{ "error": "ITEM_OUT_OF_STOCK", "itemId": "<id>" }` |
| ORD-03 | WHEN an order is submitted without a valid JWT THEN the system SHALL return `401` |
| ORD-04 | WHEN an order is submitted by a user with insufficient credit THEN the system SHALL return `422` with `{ "error": "INSUFFICIENT_CREDIT" }` |
| ORD-05 | WHEN an order is submitted with a duplicate idempotency key THEN the system SHALL return `200` with the original order ID — no new order created |

---

## Assumptions (written down)

- Payment is not charged at order creation — a separate billing service handles it asynchronously
- Stock reservation happens synchronously within this request
- An order may contain 1 to 50 line items
- Price is captured at order time — product price changes do not affect existing orders

---

## Resolved Clarifications

| Tag | Question | Resolution |
|---|---|---|
| `[NEEDS_CLARIFICATION]` | What happens to the stock reservation if the order is later cancelled? | Stock is released by a separate cancel flow — out of scope for this gate |
| `[NEEDS_CLARIFICATION]` | Is the idempotency key required or optional? | Required — reject requests without it with `400` |

---

## Do-Not List

- Do NOT charge payment within this endpoint
- Do NOT modify the product catalogue
- Do NOT touch the user account balance — only read it
- Do NOT expose internal error stack traces in responses
- Do NOT log order contents — log only order ID and user ID

---

## Spec Gate Checklist

- [x] All requirements are testable — each has specific inputs, outputs, and status codes
- [x] All assumptions are written down
- [x] All `[NEEDS_CLARIFICATION]` tags are resolved — zero open items
- [x] Do-Not list is written and reviewed
- [x] Success criteria are measurable

**Gate status: PASSED — planning may begin.**

> Next: [`plan-gate.md`](plan-gate.md)
