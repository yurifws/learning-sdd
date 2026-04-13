# PLAN_GATE.md — Plan Gate

> Validates the how before any code is written.
> The agent proposes. The human approves. Only then does implementation begin.

**Feature:** `POST /orders` — Place a new order

---

## The Rule

A plan only passes this gate when:
- It follows the project constitution (architecture rules, naming, security)
- It is simple and direct — no speculative complexity
- All contracts with other systems are defined up front
- There is a clear, local way to verify it works

---

## Proposed Plan

### Layers touched

| Layer | Component | Action |
|---|---|---|
| API | `OrderController` | New — handles `POST /orders` |
| API | `OrderOpenApi` | New — interface with `@PreAuthorize` and `@Operation` |
| Use Case | `PlaceOrderUseCase` | New — orchestrates stock check, reservation, order creation |
| Domain | `OrderDomain`, `OrderLineDomain` | New — domain models |
| Domain | `OrderPortIn` | New — inbound port |
| Persistence | `OrderPersistencePortOut` | New — outbound port |
| Persistence | `OrderPersistenceService` | New — implements outbound port |
| Persistence | `OrderRepository` | New — JPA repository |
| Persistence | `OrderEntity`, `OrderLineEntity` | New — JPA entities |
| Persistence | `StockRepository` | Existing — read and update stock |

### Flow

```
POST /orders
  → OrderController.placeOrder()
  → PlaceOrderUseCase.execute()
      → check idempotency key (OrderRepository.existsByIdempotencyKey)
        → if exists: return existing order ID with 200
      → check user credit (UserCreditPortOut.getAvailableCredit)
        → if insufficient: throw InsufficientCreditException → 422
      → for each line item:
          → StockPortOut.reserveStock(itemId, quantity)
            → if out of stock: throw OutOfStockException → 422
      → OrderPersistencePortOut.save(order)
      → return 201 with order ID
```

### Contracts with other systems

| System | Contract |
|---|---|
| `UserCreditPortOut` | `getAvailableCredit(Long userId): BigDecimal` — read-only, no side effects |
| `StockPortOut` | `reserveStock(Long itemId, int quantity): void` — throws `OutOfStockException` if unavailable |
| Billing service | Async — receives `OrderCreatedEvent` via message queue, out of scope for this gate |

---

## Constitution Compliance

- [x] Hexagonal architecture — business logic in use case, zero framework imports in domain
- [x] `@PreAuthorize` on OpenApi interface, not on controller
- [x] MapStruct for all mapping — no manual mapping
- [x] Native queries only for complex joins — simple JPA methods used here
- [x] No stack traces in responses — exception handler maps to RFC 7807 format

---

## Simplicity Check

- No caching layer added — not in requirements
- No retry logic added — billing is async and handles its own retries
- No event sourcing — simple CRUD is sufficient for this scope

---

## How to Test Locally

```bash
# 1. Start the application
./mvnw spring-boot:run

# 2. Place a valid order
curl -X POST http://localhost:8080/orders \
  -H "Authorization: Bearer <token>" \
  -H "Idempotency-Key: test-key-001" \
  -H "Content-Type: application/json" \
  -d '{"items": [{"itemId": 1, "quantity": 2}]}'
# Expected: 201 with { "orderId": "<uuid>" }

# 3. Repeat the same request
# Expected: 200 with the same orderId

# 4. Use an out-of-stock item ID
# Expected: 422 with { "error": "ITEM_OUT_OF_STOCK", "itemId": "..." }
```

---

## Plan Gate Checklist

- [x] Plan follows the project constitution
- [x] All contracts with other systems are defined
- [x] No speculative complexity — plan matches requirements exactly
- [x] A developer can run and verify this locally with the steps above
- [x] All layers and components are named before any code is written

**Gate status: PASSED — implementation may begin.**

> Next: [`TASK_GATE.md`](TASK_GATE.md)
