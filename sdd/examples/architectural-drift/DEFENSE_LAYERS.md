# Defense Layers — Catching Drift Before It Reaches Production

> The goal is not to prevent change. Change is good. Evolution is necessary. The goal is to make every change explicit — traceable and automatically enforced by a machine.

**System:** Order Service API | **Pipeline:** Pre-Merge → Pre-Deploy → Runtime

---

## The Strategy

Drift is not stopped by discipline. It is stopped by automated gates that make undocumented change impossible to ship without being detected.

The strategy is defense in depth: three independent layers, each catching what the previous layer missed.

```
Developer → [PRE-MERGE gate] → [PRE-DEPLOY gate] → [RUNTIME gate] → Production
               ↑                    ↑                    ↑
          Catch breaking        Catch contract       Catch shadow
          schema changes        mismatches between   surfaces and
          before merge          services             subtle slippage
```

A build that fails because code doesn't match the spec is not a problem. That is the system working perfectly.

---

## Layer 1 — Pre-Merge

**When it runs:** On every pull request, before a single line of new code enters the main branch.

**What it catches:** Structural drift — changes to the response shape, request schema, or status codes that contradict the OpenAPI spec.

---

### How it works

Every PR triggers an automated schema validation job. The job compares the actual API behavior (extracted from integration tests or a local test server) against the OpenAPI spec checked into the repository.

```yaml
# .github/workflows/spec-validation.yml (example)
on: [pull_request]

jobs:
  validate-spec:
    steps:
      - name: Run integration tests against local server
        run: ./gradlew integrationTest

      - name: Validate responses against OpenAPI spec
        run: npx @stoplight/spectral-cli lint openapi.yaml

      - name: Run contract tests
        run: ./gradlew contractTest
```

If the code returns `order_id` but the spec says `orderId`, the job fails. The PR cannot be merged.

---

### What it would have caught in the Order Service

| Drift | Caught by pre-merge? |
|---|---|
| `orderId` renamed to `order_id` | YES — schema validation fails on field name mismatch |
| `estimatedDelivery` added without spec update | YES — response contains undocumented field |
| Error key `ITEM_OUT_OF_STOCK` → `OUT_OF_STOCK` | YES — contract test for ORD-02 scenario fails |
| `/orders/export` added without spec update | PARTIAL — only if the route is tested; shadow routes added to internal handlers may not be covered |

---

### The rule

The OpenAPI spec lives in the repository alongside the code. It is not a wiki page, a Confluence document, or a Postman collection. It is a versioned artifact subject to the same review process as the code.

Any PR that changes API behavior must include a corresponding spec update. Any spec update requires reviewer sign-off. The CI job enforces conformance — the reviewer enforces intent.

---

## Layer 2 — Pre-Deploy

**When it runs:** After merge, before the release goes to any shared environment (staging, canary, production).

**What it catches:** Behavioral drift between services — a change in Service A that breaks Service B's expectations, even if both services individually pass their own tests.

---

### How it works

Consumer-driven contract testing. Each consumer of the Order Service (web frontend, mobile app, inventory backend) publishes a contract — a description of exactly what it expects from the API. Before any release, the Order Service must prove it satisfies every published contract.

```
Mobile app publishes:   expects GET /orders/{id} → { "order_id": ..., "status": ... }
Web frontend publishes: expects POST /orders → { "orderId": ..., "status": "CREATED" }
Inventory publishes:    expects error response { "error": "ITEM_OUT_OF_STOCK", "itemId": ... }

Pre-deploy job:
  For each consumer contract:
    1. Spin up Order Service in isolation
    2. Replay the contract's interactions against it
    3. Verify every response matches the consumer's expectation
    4. If any mismatch → FAIL → block deploy
```

This catches the case where the Order Service changed its error format from `{ "error": "ITEM_OUT_OF_STOCK" }` to `{ "error": { "code": "...", "message": "..." } }`. The inventory service contract still expects the old format. The pre-deploy check fails before anything reaches staging.

---

### What it would have caught in the Order Service

| Drift | Caught by pre-deploy? |
|---|---|
| Error key changed (`ITEM_OUT_OF_STOCK` → `OUT_OF_STOCK`) | YES — inventory service contract fails |
| Error format changed (string → object) | YES — every consumer contract that reads the error field fails |
| Response envelope added (`{ "data": { ... } }`) | YES — web frontend contract fails on field access |
| `404` changed to `500` on timeout | YES — any consumer that branches on 4xx vs 5xx fails |

---

### The rule

No service is deployed unless all consumer contracts pass. Contracts are owned by the consuming team and published to a shared contract registry. Neither team can unilaterally break the other — both must agree to any contract change before it ships.

---

## Layer 3 — Runtime

**When it runs:** Continuously, against live production traffic.

**What it catches:** Shadow surfaces (security drift), subtle behavioral slippage, and anything that passed the first two layers but diverged in real conditions.

---

### How it works

A runtime monitor samples live production traffic — a percentage of real requests and responses — and compares them against the spec. It does not replay synthetic requests. It observes what is actually happening.

```
Production traffic (sampled):
  Request: GET /orders/export?userId=123&format=csv
  Response: HTTP 200, CSV payload

Spec check:
  Is GET /orders/export in the OpenAPI spec? → NO
  Result: UNDOCUMENTED SURFACE DETECTED

Drift report:
  Route: GET /orders/export
  Status: Not in spec
  Action required: Spec update or endpoint removal — spec team notified
```

This is the layer that catches `/orders/export`. It never appeared in a PR (it was added directly). It never triggered a contract test (no consumer published a contract for it). But it showed up in production traffic. The runtime monitor saw it. The drift report named it.

---

### Evidence log format

```
Run date:    2025-07-01 | Environment: production | Sample rate: 5%

PASS    POST /orders                → 201 { "orderId": ... }         matches spec
PASS    GET  /orders/{id}           → 200 { "orderId": ... }         matches spec
DRIFT   GET  /orders/export         → 200 CSV                        NOT IN SPEC
DRIFT   POST /orders (out-of-stock) → 422 { "error": "OUT_OF_STOCK" } spec says ITEM_OUT_OF_STOCK
```

---

### What it would have caught in the Order Service

| Drift | Caught by runtime? |
|---|---|
| `/orders/export` ghost endpoint | YES — appears in sampled traffic, not in spec |
| `ITEM_OUT_OF_STOCK` → `OUT_OF_STOCK` (if it slipped through) | YES — sampled error responses don't match spec |
| `legacyOrderRef` still being returned | YES — undocumented field in sampled responses |
| Auth accepting API keys without spec update | YES — sampled requests with API keys succeed; spec says Bearer-only |

---

### The rule

If a mismatch is found at runtime, the spec is updated before the code is changed. Always.

The observed behavior may be correct (the spec was wrong or outdated). The spec may be correct (the code regressed). The decision about which is which requires a human. But the spec is updated first to make that decision explicit, visible, and version-controlled — not buried in a ticket or lost in Slack.

See [`../verification-gates/RUNTIME_GATE.md`](../verification-gates/RUNTIME_GATE.md) for a worked example of this resolution process.

---

## The Full Pipeline

```
Change made → PR opened
                 ↓
          [PRE-MERGE GATE]
          Schema validation
          Contract tests
          Spec diff review
                 ↓ passes
             PR merged
                 ↓
          [PRE-DEPLOY GATE]
          Consumer contract tests
          Cross-service validation
                 ↓ passes
           Deploy to staging → Production
                 ↓
           [RUNTIME GATE]
           Live traffic sampling
           Undocumented surface detection
           Continuous spec conformance check
                 ↓ drift detected
           Spec update required → back to PRE-MERGE
```

---

## Failing Fast Is Not Slowing Down

The three-layer pipeline feels like friction. It is not.

Without it, every undetected drift accumulates silently. The cost of catching drift in production — an incident, a breach, a customer-facing bug — is orders of magnitude higher than the cost of a failed CI job.

Failing fast means catching the inevitable changes early, when the cost to fix is low and the change is still fresh in the developer's mind.

| Where drift is caught | Cost to fix |
|---|---|
| Pre-merge (spec diff review) | Minutes — developer updates the spec alongside the code |
| Pre-deploy (contract test failure) | Hours — two teams align on the new contract |
| Runtime (production monitoring) | Days — incident, root cause, hotfix, post-mortem |
| Never (undetected) | Breach, data loss, cascading failure |

The pipeline does not slow delivery. It makes delivery safe.

---

## Checklist — Drift Defense for Your System

### Pre-Merge
- [ ] OpenAPI spec is committed to the repository alongside code
- [ ] CI validates API responses against the spec on every PR
- [ ] Any PR that changes API behavior must include a spec update
- [ ] Spec changes require reviewer sign-off

### Pre-Deploy
- [ ] Each consumer service publishes a contract to a shared registry
- [ ] Pre-deploy job runs all consumer contracts against the service before any release
- [ ] No deployment proceeds if any consumer contract fails

### Runtime
- [ ] Production traffic is sampled and compared to the spec continuously
- [ ] Alerts fire on any undocumented surface or response mismatch
- [ ] Any detected drift triggers a spec update before a code change
- [ ] The spec remains the system of record after every run

**Gate status: All three layers active → drift is caught, not accumulated.**
