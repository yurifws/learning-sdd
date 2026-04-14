# CONTRACT_FLOW.md

> Most multi-agent bugs don't happen inside an agent's work.
> They happen at the seam — where one agent's output meets another agent's input.
> Contracts are what you put at that seam.

---

## The Seam Problem

When two agents (or two developers) work on adjacent parts of a system, one thing breaks the collaboration more than anything else: **each side assumes what the other is building.**

The UI team builds against the API endpoint they expected. The API team changed the shape of that endpoint. Neither noticed. The bug ships.

The classic failure modes:
- UI builds a feature for an API contract that has already changed
- API expects a database field that was renamed or removed
- Two agents touch the same file and overwrite each other's work

None of these failures happen *inside* an agent. They all happen *between* agents. That's the seam.

---

## What a Contract Is

A contract is a small, structured artifact that:
1. One agent **produces** as the output of its work
2. A downstream agent **consumes** as the input to its work
3. Is **immutable** once the downstream agent has started — it cannot change without stopping and re-coordinating

Contracts live in markdown files. They are written before any implementation code.

---

## The Contract Chain

Each layer in a system produces a contract and consumes one from upstream:

```
┌─────────────────────────────────────────────────────────┐
│  DB Agent                                               │
│  Input:  requirements.md (schema rules)                 │
│  Output: db-contract.md  (table names, column types,    │
│                            FK relationships, indexes)   │
└────────────────────────┬────────────────────────────────┘
                         │ reads db-contract.md
┌────────────────────────▼────────────────────────────────┐
│  API Agent                                              │
│  Input:  db-contract.md + requirements.md               │
│  Output: api-contract.md  (endpoints, request/response  │
│                             shapes, status codes,       │
│                             auth rules)                 │
└────────────────────────┬────────────────────────────────┘
                         │ reads api-contract.md
┌────────────────────────▼────────────────────────────────┐
│  UI Agent                                               │
│  Input:  api-contract.md + design.md                   │
│  Output: implemented UI components                      │
│  (no contract produced — UI is the terminal layer)      │
└─────────────────────────────────────────────────────────┘
```

The code must follow the contract. The contract is never changed to match what the code happens to produce.

---

## What a Contract File Looks Like

### `db-contract.md`

```markdown
# DB Contract — Orders Feature
# Status: LOCKED (API agent has started)

## Table: orders

| Column | Type | Constraints |
|---|---|---|
| id | UUID | PRIMARY KEY |
| user_id | BIGINT | NOT NULL, FK → users.id |
| idempotency_key | VARCHAR(64) | UNIQUE, NOT NULL |
| status | VARCHAR(32) | NOT NULL, CHECK IN ('PENDING','CONFIRMED','CANCELLED') |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT now() |

## Table: order_lines

| Column | Type | Constraints |
|---|---|---|
| id | BIGINT | PRIMARY KEY, GENERATED |
| order_id | UUID | NOT NULL, FK → orders.id ON DELETE CASCADE |
| item_id | BIGINT | NOT NULL |
| quantity | INT | NOT NULL, CHECK > 0 |
| unit_price | NUMERIC(10,2) | NOT NULL |
```

### `api-contract.md`

```markdown
# API Contract — Orders Feature
# Status: LOCKED (UI agent has started)

## POST /orders

**Auth:** Bearer JWT, requires ROLE_PLACE_ORDER
**Idempotency-Key header:** required, max 64 chars

### Request body
{
  "items": [{ "itemId": number, "quantity": number }]
}

### Responses
| Status | When | Body |
|---|---|---|
| 201 | Order created | `{ "orderId": "uuid" }` |
| 200 | Duplicate idempotency key | `{ "orderId": "uuid" }` (same as original) |
| 422 | Insufficient credit | `{ "error": "INSUFFICIENT_CREDIT" }` |
| 422 | Item out of stock | `{ "error": "ITEM_OUT_OF_STOCK", "itemId": number }` |
| 401 | Missing or invalid JWT | standard 401 |
```

---

## The Rules

**1. Write the contract before writing code.**
The contract is the design artifact. Code is just the implementation of it.

**2. Lock the contract before the downstream agent starts.**
Add `# Status: LOCKED` to the file header. Any change to a locked contract requires stopping downstream agents and coordinating a re-plan.

**3. Agents read upstream contracts; they do not reach past them.**
The UI agent reads `api-contract.md`. It never reads `db-contract.md`. Agents do not skip layers.

**4. The task format enforces the contract boundary.**
Each task in `tasks.md` lists its `Files to Touch`. That list is the file-level contract — an agent that touches a file outside its list has violated its boundary.

```markdown
## Task N — [Name]
**Agent role:** Persistence Specialist
**Reads:** db-contract.md
**Produces:** (updates api-contract.md with persistence port signatures)
**Files to touch:**
  - src/.../OrderEntity.java
  - src/.../OrderRepository.java
  - src/.../OrderPersistenceService.java
**Must NOT touch:** any controller or use case file
**Verify:** ./mvnw test -Dtest=OrderPersistenceServiceTest
```

---

## Connecting to the Orchestrator

The orchestrator agent's job is to distribute contracts, not to hold them:

```
Orchestrator reads:
  PROJECT_PLAN.md, PROJECT_TECHSTACK.md, tasks.md

Orchestrator distributes:
  → DB Agent:  db-contract.md (to produce) + requirements.md
  → API Agent: api-contract.md (to produce) + db-contract.md (to consume)
  → UI Agent:  api-contract.md (to consume) + design.md

Orchestrator verifies:
  Runs acceptance tests after each agent reports done.
  Stops all work if any agent reports [NEEDS_CLARIFICATION].
```

The orchestrator never implements. It coordinates, distributes, and verifies.

---

## When This Matters Most

| Situation | Why contracts matter |
|---|---|
| 2+ agents working in parallel | Without a shared contract, each agent invents the interface independently |
| Layered architecture (hexagonal, layered MVC) | Each layer boundary is a natural contract point |
| Backend + frontend teams | The API contract is the only shared truth — without it, both sides drift |
| Long-running features (multiple sessions) | A locked contract survives session resets; a verbal agreement does not |

See [`ORCHESTRATION.md`](ORCHESTRATION.md) for the full orchestrator and specialist prompt templates.
