# TASK_GATE.md — Task Gate

> Breaks the approved plan into chunks small enough for a human to review
> in a single focused session. Every task is atomic, verifiable, and traceable.

**Feature:** `POST /orders` — Place a new order

---

## The Rule

A task only passes this gate when:
- It is **atomic** — does exactly one thing
- It is **verifiable** — has a specific test to run to confirm it is done
- It is **traceable** — links back to the plan section and the spec requirement it fulfills

If you cannot confidently say "this is correct" after one focused review session, the task is too big.

---

## Task List

| # | Task | Traces to | Status |
|---|---|---|---|
| 1 | Create `OrderEntity` and `OrderLineEntity` JPA entities | Plan: Persistence layer | ⏳ |
| 2 | Create `OrderRepository` with `existsByIdempotencyKey` | Plan: Persistence layer | ⏳ |
| 3 | Create `OrderDomain`, `OrderLineDomain` | Plan: Domain layer | ⏳ |
| 4 | Create `OrderPortIn` and `OrderPersistencePortOut` ports | Plan: Domain / Persistence | ⏳ |
| 5 | Create `PlaceOrderUseCase` — idempotency check only | ORD-05 | ⏳ |
| 6 | Extend `PlaceOrderUseCase` — credit check | ORD-04 | ⏳ |
| 7 | Extend `PlaceOrderUseCase` — stock reservation | ORD-02 | ⏳ |
| 8 | Create `OrderPersistenceService` — save and retrieve order | ORD-01 | ⏳ |
| 9 | Create `OrderController` and `OrderOpenApi` | ORD-01, ORD-03 | ⏳ |
| 10 | Integration test — full happy path | ORD-01 | ⏳ |

---

## Active Task — Task 5

**Task:** Create `PlaceOrderUseCase` — idempotency check only

**Traces to:**
- Plan: `PlaceOrderUseCase.execute()` → idempotency check branch
- Spec: ORD-05 — duplicate idempotency key returns `200` with original order ID

**Atomic scope:** Only the idempotency branch. No stock check. No credit check. No order creation.

---

### Files to Touch

| File | Action |
|---|---|
| `PlaceOrderUseCase.java` | Create — implement idempotency check, delegate to `OrderPersistencePortOut` |
| `OrderPersistencePortOut.java` | Add `findByIdempotencyKey(String key): Optional<OrderDomain>` |
| `PlaceOrderUseCaseTest.java` | Create — tests for idempotency branch only |

---

### Done When

- [ ] `PlaceOrderUseCase` returns existing order ID when idempotency key already exists
- [ ] `PlaceOrderUseCase` proceeds (does not throw) when idempotency key is new
- [ ] Unit test for duplicate key passes
- [ ] Unit test for new key passes
- [ ] No other logic exists in this class yet — stock and credit checks are not implemented

---

### Test

```java
@Test
void execute_duplicateIdempotencyKey_returnsExistingOrderId() {
    // given
    String key = "idem-key-001";
    OrderDomain existing = OrderDomain.builder().id(UUID.randomUUID()).build();
    when(orderPersistencePortOut.findByIdempotencyKey(key)).thenReturn(Optional.of(existing));

    // when
    PlaceOrderResult result = placeOrderUseCase.execute(PlaceOrderCommand.builder()
        .idempotencyKey(key)
        .build());

    // then
    assertThat(result.isIdempotent()).isTrue();
    assertThat(result.getOrderId()).isEqualTo(existing.getId());
    verify(orderPersistencePortOut, never()).save(any());
}

@Test
void execute_newIdempotencyKey_proceeds() {
    // given
    String key = "idem-key-new";
    when(orderPersistencePortOut.findByIdempotencyKey(key)).thenReturn(Optional.empty());

    // when — no exception thrown
    assertThatCode(() -> placeOrderUseCase.execute(PlaceOrderCommand.builder()
        .idempotencyKey(key)
        .build()))
        .doesNotThrowAnyException();
}
```

---

## Task Gate Checklist (per task)

- [x] Task does exactly one thing
- [x] Test exists that confirms the task is done
- [x] Task traces back to a plan section and a spec requirement
- [x] A reviewer can confirm correctness in one focused session

**Gate status: PASSED — implementation of Task 5 may begin.**

> Next: [`RUNTIME_GATE.md`](RUNTIME_GATE.md)
