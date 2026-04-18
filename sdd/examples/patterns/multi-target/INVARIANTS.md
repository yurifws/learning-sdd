# Invariants — What Cannot Change Across Targets

> The entire multi-target model rests on one discipline: ruthless clarity about what is allowed to differ between implementations, and what absolutely cannot.

**System:** Payment Processing Service | **Targets:** Go (primary) · Rust (high-throughput)

---

## The Foundation

A target is not a free implementation of the spec. It is a constrained one.

The constraint is the invariant set: the explicit, tested list of behaviors, contracts, and quality thresholds that must hold identically across every target, forever. An invariant is not a guideline. It is a compile-time requirement. A target that violates an invariant is not a valid target.

There are three classes.

---

## Class 1 — Functional Invariants

**What they are:** The core behaviors of the system. The user-visible outcomes that must be identical regardless of which target handles the request.

**The rule:** If behavior X is true for the Go service, it must be true for the Rust service. No exceptions. Any change to a functional invariant must be made to the spec first and then regenerated to all targets simultaneously.

---

### Payment Service functional invariants

```
FINV-01  WHEN a payment request arrives with a valid idempotency key
         that was previously used for a completed payment,
         THEN the system SHALL return the original payment result
         AND SHALL NOT create a new charge.

FINV-02  WHEN a payment request arrives with an amount of zero or less,
         THEN the system SHALL return 422 with error code INVALID_AMOUNT
         AND SHALL NOT attempt to contact the payment provider.

FINV-03  WHEN a payment provider returns a decline code,
         THEN the system SHALL return 402 with the decline reason
         AND SHALL NOT retry automatically.

FINV-04  WHEN a payment succeeds,
         THEN the system SHALL record the transaction with status COMPLETED
         AND the recorded amount SHALL exactly equal the requested amount.

FINV-05  WHEN any internal error occurs during payment processing,
         THEN the system SHALL return 500 with error code PAYMENT_PROCESSING_ERROR
         AND SHALL NOT expose provider error details, stack traces, or internal identifiers.
```

---

**What differs between targets — and what does not:**

| Aspect | Go implementation | Rust implementation | Changes allowed? |
|---|---|---|---|
| HTTP framework | `net/http` + `chi` | `axum` + `tokio` | ✅ Yes — implementation detail |
| Concurrency model | Goroutines | Async/await + Tokio runtime | ✅ Yes — implementation detail |
| Idempotency behavior (FINV-01) | Redis-backed key check | Redis-backed key check | ❌ No — functional invariant |
| Zero-amount rejection (FINV-02) | Input validation middleware | Request guard | ❌ No — functional invariant |
| Error response shape (FINV-05) | JSON encoder | `serde` serializer | ❌ No — functional invariant |

The implementation machinery is free to differ. The behavior is not.

---

## Class 2 — Contract Invariants

**What they are:** The shape of the system's interface with the outside world. Every field name, status code, error code, and header must be identical across all targets. Consumers must be able to switch targets without changing their integration.

**The rule:** The API contract is an OpenAPI spec. It is frozen before any implementation begins. No target may deviate from it. A target that returns `payment_id` where the contract says `paymentId` is a broken target — even if everything else works.

---

### Payment Service contract invariants

The OpenAPI spec is the source of truth. Key contract clauses:

```yaml
# openapi.yaml — frozen before any target is implemented

POST /payments:
  requestBody:
    required: [amount, currency, source, idempotencyKey]
    properties:
      amount:         { type: integer, description: "Amount in smallest currency unit" }
      currency:       { type: string, enum: [USD, EUR, GBP] }
      source:         { type: string, description: "Payment method token" }
      idempotencyKey: { type: string, maxLength: 64 }

  responses:
    201:
      properties:
        paymentId:  { type: string, format: uuid }
        status:     { type: string, enum: [COMPLETED, PENDING] }
        amount:     { type: integer }
        currency:   { type: string }
        createdAt:  { type: string, format: date-time }
    422:
      properties:
        error:    { type: string }   # INVALID_AMOUNT | INVALID_CURRENCY | MISSING_FIELD
        field:    { type: string }
    402:
      properties:
        error:    { type: string }   # CARD_DECLINED | INSUFFICIENT_FUNDS | EXPIRED_CARD
        declineCode: { type: string }
```

---

**Contract invariant checklist — verified identically for both targets:**

- [ ] `POST /payments` accepts exactly the fields defined in the contract — no extras, no missing
- [ ] `201` response includes `paymentId`, `status`, `amount`, `currency`, `createdAt` — all present, all typed correctly
- [ ] `422` error codes match exactly: `INVALID_AMOUNT`, `INVALID_CURRENCY`, `MISSING_FIELD` — no variants
- [ ] `402` includes `error` and `declineCode` — no additional fields that could leak provider details
- [ ] Idempotency header is `Idempotency-Key` — exact casing, not `idempotency-key` or `X-Idempotency-Key`

A consumer who calls the Go service and then the Rust service must receive responses they cannot distinguish. The contract is the guarantee.

---

## Class 3 — Quality Invariants

**What they are:** The performance and reliability thresholds the system must meet. A Rust implementation that violates the API contract is broken. A Rust implementation that meets the API contract but degrades to 2,000ms P95 latency — when the spec says 200ms — is also broken.

**The rule:** Quality invariants are measured using a shared benchmark harness with identical load profiles, identical test data, and identical measurement methodology. The targets are not compared to each other. Each is compared independently to the spec threshold.

---

### Payment Service quality invariants

```
QINV-01  UNDER a load of 1,000 requests/second sustained for 60 seconds,
         the system SHALL maintain P95 latency below 200ms
         as measured from request receipt to response completion.

QINV-02  UNDER normal load (≤ 500 requests/second),
         the system SHALL maintain an error rate below 0.1%
         excluding payment provider declines (which are correct behavior).

QINV-03  WHEN the payment provider is unreachable,
         the system SHALL return a 503 response within 5,000ms
         (circuit breaker or timeout — not an indefinite hang).

QINV-04  The system SHALL handle a burst of 3,000 requests/second for 10 seconds
         without dropping requests or returning 5xx responses.
```

---

**Benchmark harness — shared across both targets:**

```
Harness: k6 (identical script for both targets)
Test data: 10,000 pre-seeded payment tokens — same dataset for both
Measurement: external timing only (wall clock, not instrumentation)
Environment: identical hardware spec (4 vCPU, 8GB RAM, same network tier)
Warm-up: 30 seconds at 100 req/s before measurement begins
Run: 60 seconds at target load
Metrics: P50, P95, P99 latency; error rate; throughput
```

The Go service running on different hardware than the Rust service is not a fair comparison. Both must run on identical infrastructure, measured by the same tool, with the same data. Otherwise you are not comparing implementations — you are comparing environments.

---

## The Invariant Boundary

The invariant boundary is the explicit line between what is controlled by the spec and what is free for each target to decide.

```
CONTROLLED BY SPEC (identical across all targets)
─────────────────────────────────────────────────
  Functional: idempotency, error handling, financial correctness, security
  Contract:   field names, status codes, error codes, headers
  Quality:    P95 latency, error rate, circuit breaker timeout
─────────────────────────────────────────────────
FREE PER TARGET (implementation choice)
─────────────────────────────────────────────────
  HTTP framework and routing library
  Concurrency model (goroutines, async/await, thread pool)
  Internal data structures and caching strategy
  Dependency injection approach
  Logging and metrics format (as long as they don't appear in responses)
  Test framework
```

Anything above the line is enforced by the spec and verified by the shared test harness. Anything below the line is the target author's decision. A target that treats something below the line as optional is a valid target. A target that treats something above the line as optional is a broken target.

---

## Invariant Enforcement Checklist

Before any target is accepted as valid:

- [ ] All functional invariants (FINV-01 through FINV-05) have passing contract tests
- [ ] All contract invariants are verified by running the OpenAPI contract test suite against the target
- [ ] All quality invariants have benchmark results from the shared harness meeting the defined thresholds
- [ ] No target-specific behavior has been introduced that is not in the spec
- [ ] Any observed behavioral difference from another target has been traced to a spec clause — not left as "implementation variance"

**A target is not valid until all three invariant classes pass. Passing two out of three is a broken target.**
