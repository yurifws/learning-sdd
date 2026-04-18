# Generation Pipeline — From One Spec to Multiple Targets

> Maintenance is no longer about patching two codebases. It is about updating the spec and regenerating. The spec is the real source code. The code is emitted.

**System:** Payment Processing Service | **Targets:** Go (primary) · Rust (high-throughput)

---

## The Four-Step Process

```
Step 1: Write a language-agnostic spec
           ↓
Step 2: Freeze the API contract
           ↓
Step 3: Write separate implementation plans per target
           ↓
Step 4: Benchmark both targets with a shared harness
```

Each step is a gate. Step 2 cannot begin until Step 1 is locked. Step 3 cannot begin until Step 2 is frozen. A target is not accepted until Step 4 passes. This order is non-negotiable — it is what prevents the two targets from diverging before they are even written.

---

## Step 1 — Write a Language-Agnostic Spec

The spec must express **pure intent**. No library names. No language-specific constructs. No implementation assumptions. If the spec mentions `goroutines`, `tokio`, `CompletableFuture`, or any framework, it is not a spec — it is a partial implementation plan leaking upward.

---

**Spec for the Payment Processing Service**

Written in EARS. Every clause is testable without knowing which target will execute it.

```
PAY-01  WHEN a POST /payments request arrives with all required fields
        AND the idempotency key has not been used before,
        THEN the system SHALL process the payment with the configured provider
        AND SHALL return 201 with the payment result.

PAY-02  WHEN a POST /payments request arrives with an idempotency key
        that was previously used for a completed payment,
        THEN the system SHALL return the original 201 response
        AND SHALL NOT contact the payment provider again.

PAY-03  WHEN the amount field is zero or negative,
        THEN the system SHALL return 422 with error code INVALID_AMOUNT
        AND SHALL NOT contact the payment provider.

PAY-04  WHEN the payment provider returns a decline,
        THEN the system SHALL return 402 with the decline code
        AND SHALL NOT retry.

PAY-05  WHEN the payment provider is unreachable for longer than 4,500ms,
        THEN the system SHALL return 503 with error code PROVIDER_UNAVAILABLE.

PAY-06  The system SHALL never include provider error messages, stack traces,
        or internal identifiers in any response body.
```

These clauses say nothing about how to implement idempotency (Redis? DB? In-memory?), how to enforce the timeout (circuit breaker? deadline propagation?), or how to serialize the response. Those are target decisions.

---

## Step 2 — Freeze the API Contract

The API contract is an OpenAPI spec. It is agreed on and committed before any implementation begins. After this step, no target may deviate from the contract. Any required change triggers a version bump and a new round of cross-target validation.

```yaml
# openapi.yaml — locked at this step, read-only for both targets

openapi: "3.0.3"
info:
  title: Payment Processing Service
  version: "1.0.0"

paths:
  /payments:
    post:
      operationId: createPayment
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/PaymentRequest'
      responses:
        '201': { $ref: '#/components/responses/PaymentCreated' }
        '402': { $ref: '#/components/responses/PaymentDeclined' }
        '422': { $ref: '#/components/responses/ValidationError' }
        '503': { $ref: '#/components/responses/ProviderUnavailable' }

components:
  schemas:
    PaymentRequest:
      required: [amount, currency, source, idempotencyKey]
      properties:
        amount:         { type: integer, minimum: 1 }
        currency:       { type: string, enum: [USD, EUR, GBP] }
        source:         { type: string }
        idempotencyKey: { type: string, maxLength: 64 }
```

The contract defines what is observable. Both targets are built to this contract — not to each other.

---

## Step 3 — Write Per-Target Implementation Plans

This is where the targets diverge — intentionally and explicitly. Each target gets its own implementation plan. The plans describe the how. The spec describes the what. They must never contradict each other.

---

### Plan A — Go (primary service)

```markdown
## Payment Service — Go Implementation Plan

### Stack
- Runtime: Go 1.22
- HTTP router: chi v5
- Idempotency store: Redis (go-redis/v9)
- Provider client: custom HTTP wrapper with context deadlines
- Serialization: encoding/json (standard library)
- Concurrency: goroutines with sync.WaitGroup for provider calls

### PAY-02 — Idempotency implementation
1. Extract Idempotency-Key header from request
2. Attempt SET NX (atomic Redis set-if-not-exists) with 24h TTL
3. If key exists: fetch cached response from Redis, return directly
4. If key is new: proceed with payment, store result in Redis after 201

### PAY-05 — Provider timeout implementation
1. Attach context.WithTimeout(ctx, 4500*time.Millisecond) to provider call
2. On context.DeadlineExceeded: return 503 PROVIDER_UNAVAILABLE immediately
3. No retry — PAY-04 prohibits automatic retries

### Quality targets (from QINV-01 through QINV-04)
- P95 < 200ms: achieved via goroutine-per-request model + connection pooling
- Error rate < 0.1%: achieved via structured error handling, no panic propagation
```

---

### Plan B — Rust (high-throughput service)

```markdown
## Payment Service — Rust Implementation Plan

### Stack
- Runtime: Rust 1.77 + Tokio async runtime
- HTTP framework: axum 0.7
- Idempotency store: Redis (redis-rs with connection pooling)
- Provider client: reqwest with timeout configuration
- Serialization: serde_json
- Concurrency: async/await task model

### PAY-02 — Idempotency implementation
1. Extract Idempotency-Key header via axum extractor
2. Attempt Redis SET NX via async connection pool
3. If key exists: deserialize cached response, return directly
4. If key is new: process payment, store result in Redis after 201
   (identical semantics to Go plan — different async machinery)

### PAY-05 — Provider timeout implementation
1. Configure reqwest client with timeout(Duration::from_millis(4500))
2. On Elapsed error: return 503 PROVIDER_UNAVAILABLE
3. No retry — same PAY-04 constraint

### Quality targets (from QINV-01 through QINV-04)
- P95 < 200ms: expected to be significantly under target due to zero-cost abstractions
- Error rate < 0.1%: achieved via Result<T, E> propagation, no unwrap() in request path
```

---

**The key observation:** The implementation plans use completely different technologies, concurrency models, and async primitives. The behavioral outcomes they describe — idempotency semantics, timeout behavior, error codes — are identical. The what comes from the spec. The how is the plan's domain.

---

## Step 4 — Benchmark with a Shared Harness

Both targets are validated against the same quality invariants using the same harness. The purpose is not to compare Go against Rust. The purpose is to confirm that each target independently meets the spec thresholds.

```javascript
// k6-payment-benchmark.js — identical for both targets, only BASE_URL differs

import http from 'k6/http';
import { check } from 'k6';

export const options = {
  stages: [
    { duration: '30s', target: 1000 },  // warm-up
    { duration: '60s', target: 1000 },  // sustained load — QINV-01
    { duration: '10s', target: 3000 },  // burst — QINV-04
    { duration: '10s', target: 0   },   // wind-down
  ],
  thresholds: {
    http_req_duration: ['p(95)<200'],   // QINV-01
    http_req_failed:   ['rate<0.001'],  // QINV-02
  },
};

export default function () {
  const res = http.post(`${__ENV.BASE_URL}/payments`, JSON.stringify({
    amount: 5000,
    currency: 'USD',
    source: `tok_${Math.random().toString(36).slice(2)}`,
    idempotencyKey: `test-${__VU}-${__ITER}`,
  }), { headers: { 'Content-Type': 'application/json' } });

  check(res, {
    'status is 201':          r => r.status === 201,
    'paymentId present':      r => r.json('paymentId') !== undefined,
    'no provider details':    r => !JSON.stringify(r.body).includes('stripe'),
  });
}
```

Both targets run this script on identical infrastructure. Both must pass. A target that passes the contract tests but fails the benchmark is not a valid target.

---

## UI Variants — Functional vs. Aesthetic

The same pipeline applies to user interfaces. The discipline is the same: separate what is functional (controlled by spec) from what is aesthetic (free per target).

```
CONTROLLED BY SPEC (identical across all UI variants)
──────────────────────────────────────────────────────
  Core user flows:          "Place order" → confirm → receipt
  Validation rules:         Required fields, format checks, error messages
  Accessibility:            WCAG 2.1 AA compliance, keyboard navigation
  State transitions:        Loading, success, error, empty states

FREE PER VARIANT (implementation choice)
──────────────────────────────────────────────────────
  Layout and spacing
  Color palette and typography
  Component library (Material, Radix, custom)
  Animation and motion
  Brand-specific iconography
```

A "consumer" skin and an "enterprise" skin are two valid targets of the same UI spec. The flows are identical. The aesthetics are entirely different.

---

### Figma as design source of truth

The same principle that makes the OpenAPI spec the contract source of truth for services applies to design: if the design intent lives only in someone's head or in a static screenshot, every UI variant will interpret it differently.

By pulling design tokens and component structures directly from Figma's API into the generation pipeline, every UI variant is built from the actual intended design — not from someone's interpretation of it.

```
Figma (design intent)
      ↓
Design tokens extracted via Figma API
  (colors, spacing, typography, component states)
      ↓
Token file committed to repository
      ↓
UI variants consume the token file — never hardcode visual values
```

A color that changes in Figma propagates to every variant on the next generation run. No manual update. No variant that missed the memo.

---

## Determinism Engineering

The pipeline is only as reliable as the stability of its three layers.

### Stable inputs
- [ ] The spec is version-controlled and tagged — every generation is traceable to a spec version
- [ ] All dependencies are pinned — no floating versions that change between runs
- [ ] The OpenAPI contract is committed alongside the spec — not generated dynamically

### Stable pipelines
- [ ] Generation steps are deterministic — same inputs always produce the same outputs
- [ ] Build steps are containerized — same environment every time, not "works on my machine"
- [ ] Contract test suite is version-locked alongside the contract

### Stable outputs
- [ ] Every artifact is tagged with the spec version that produced it
- [ ] The relationship is traceable: spec version 1.3.2 → artifact build #447 → Go service v1.3.2
- [ ] Rolling back an artifact means rolling back to a known spec version, not guessing

```
Spec v1.3.2 (tagged)
      ↓
Contract test suite v1.3.2 (locked to spec version)
      ↓
Go service build #447   ←→   Rust service build #447
(both tagged spec:1.3.2)      (both tagged spec:1.3.2)
```

If a build cannot be traced to a specific spec version, it is not a valid build. Determinism is not a property of the output — it is a property of the pipeline that produced it.

---

## The Question Code Can't Answer

The spec-driven multi-target model inverts the traditional value hierarchy.

In the traditional model: code is the primary asset. The spec is documentation. The spec may or may not reflect reality.

In this model: the spec is the primary asset. The code is a derived artifact — valuable, but replaceable. If the Go service needs to be rewritten in a new language, the spec survives. The invariants survive. The contract survives. Only the implementation is discarded.

> If the spec is the most valuable asset in the system, what does that make the code?

It makes it an artifact. One that can be regenerated, replaced, or discarded — as long as the spec, the contracts, and the invariants are intact.

That is the whole point.
