# tasks.md — Development Tasks

> Ordered task list for the Client API project.
> One task at a time. Stop after each task and wait for approval.
> The **Active Task** section below is the only part you update constantly.

---

## Instructions for Claude

Read `AGENTS.md` before starting. Then:
1. Find the first unchecked task in the current phase.
2. Read the **Active Task** section — it has the acceptance criteria and exact files to touch.
3. After each task: run `./mvnw test`, check done criteria, update `plan.md`, wait for approval.

---

## Active Task

> This is the only section you update constantly.
> Change "Active Task" and "Files to Touch" only when starting something new.
> Keep CONSTITUTION.md, spec.md, and plan.md stable unless a contract or model actually changed.

### When this task is done
1. Every box in § Done When must be `[x]`
2. Run `./mvnw test` — all tests green
3. Update `plan.md` — mark endpoint ✅ Done and update test columns
4. Update `spec.md` — only if a contract, model, or layer pattern changed
5. Clear this section — write the next task here

---

**Task:** Implement `GET /clients/{idClient}/addresses` — Find client addresses

### Acceptance Criteria
- Authenticated users with `ROLE_CLIENT_QUERY_ADDRESS` can call this endpoint
- Returns a list of addresses (`List<AddressResponseModel>`) for the given client ID
- Returns `404` with a clear message if the client does not exist
- Returns `200` with an empty list if the client exists but has no registered addresses

### Context
- The client summary flow is already implemented — follow the exact same pattern (see `spec.md`)
- `ClientOpenApi` interface already exists — add the new method there
- `ClientController` already exists — add the new method there
- New role constant must be added to `Authorities`

### Files to Create / Touch

| File | Action |
|---|---|
| `Authorities.java` | Add `ROLE_CLIENT_QUERY_ADDRESS` constant |
| `ClientPortIn.java` | Add `findAddresses(Long idClient)` method |
| `ClientUseCase.java` | Implement `findAddresses` |
| `ClientPersistencePortOut.java` | Add `findAddresses(Long idClient)` method |
| `ClientPersistenceService.java` | Implement `findAddresses` with repository call |
| `AddressQueryResultDTO.java` | New: query result projection interface |
| `AddressDomain.java` | New: domain model |
| `AddressResponseModel.java` | New: response record with `@Schema` |
| `ClientPersistenceMapper.java` | Add `toDomain(AddressQueryResultDTO)` |
| `ClientControllerMapper.java` | Add `toResponse(AddressDomain)` |
| `ClientRepository.java` | Add `findAddressesByClientId(@Param Long idClient)` |
| `ClientOpenApi.java` | Add `findAddresses` method signature |
| `ClientController.java` | Add `findAddresses` implementation |

### Done When
- [ ] Endpoint returns `200` with list of addresses for a valid client
- [ ] Endpoint returns `200` with empty list when client has no addresses
- [ ] Endpoint returns `404` when client ID does not exist
- [ ] `@PreAuthorize` is set correctly on the OpenApi method
- [ ] All layers follow the hexagonal pattern from `spec.md`
- [ ] All mappers use MapStruct (no manual mapping)
- [ ] Logs present at controller, usecase, and persistence levels

### Notes / Blockers
- [ ] `[NEEDS_CLARIFICATION]` Define the fields for `AddressResponseModel` before starting (street, number, neighborhood, city, state, zipCode?)
- [ ] `[NEEDS_CLARIFICATION]` Confirm if address table has a FK to the client table or uses a join table

---

## Phase 1 — Foundation

- [x] **TASK-01** — Set up Spring Boot project with Hexagonal package structure
  **Verify:** `./mvnw clean install` passes; package tree matches `spec.md`

- [x] **TASK-02** — Create `BaseEntity`, `BaseMapper`, `Authorities`, OpenAPI base config
  **Verify:** App starts; Swagger UI loads at `/swagger-ui.html`

- [x] **TASK-03** — Configure Spring Security with JWT filter and role-based access
  **Verify:** Request without token returns HTTP 401

---

## Phase 2 — GET /clients/{idClient}/summary

- [x] **TASK-04** — Write tests for the summary endpoint (ADDR-S1 through ADDR-S4)
  **Verify:** All tests FAIL before any implementation

- [x] **TASK-05** — Add `findSummaryById` to `ClientPortIn` and `ClientPersistencePortOut`
  **Verify:** Project compiles

- [x] **TASK-06** — Implement `ClientUseCase.findSummaryById`
  **Verify:** Unit test for use case passes

- [x] **TASK-07** — Implement `ClientPersistenceService.findSummaryById` with native query
  **Verify:** Integration test against H2 passes

- [x] **TASK-08** — Add `GET /clients/{idClient}/summary` to `ClientOpenApi` and `ClientController`
  **Verify:** All summary endpoint tests pass; `./mvnw test` green

---

## Phase 3 — GET /clients/{idClient}/addresses

- [ ] **TASK-09** — Resolve clarification gate (see `CLARIFICATION_GATE.md`)
  **Verify:** All 5 ambiguity buckets answered; no `[NEEDS_CLARIFICATION]` remaining

- [ ] **TASK-10** — Write tests for the addresses endpoint (ADDR-T1 through ADDR-T7)
  **Verify:** All 7 tests FAIL — see `TEST_FIRST_GATE.md` for evidence

- [ ] **TASK-11** — Add `AddressDomain` and `AddressResponseModel`
  **Verify:** Project compiles

- [ ] **TASK-12** — Add `findAddresses` to `ClientPortIn` and `ClientPersistencePortOut`
  **Verify:** Project compiles

- [ ] **TASK-13** — Add `ROLE_CLIENT_QUERY_ADDRESS` to `Authorities`
  **Verify:** Project compiles; new role constant visible in Swagger UI

- [ ] **TASK-14** — Implement `ClientUseCase.findAddresses`
  **Verify:** ADDR-T6 passes (`ClientUseCaseTest`)

- [ ] **TASK-15** — Implement `ClientPersistenceService.findAddresses` with native query
  **Verify:** ADDR-T7 passes (`ClientPersistenceServiceTest`)

- [ ] **TASK-16** — Add `findAddresses` to `ClientOpenApi` and `ClientController` with MapStruct mapper
  **Verify:** ADDR-T1 through ADDR-T5 pass; `./mvnw test` green

- [ ] **TASK-17** — Update `plan.md` — mark addresses endpoint ✅ Done
  **Verify:** plan.md reflects current status; no drift between spec and implementation

---

## Phase 4 — Pending Endpoints (Next)

- [ ] **TASK-18** — GET /clients/{idClient}/contacts
- [ ] **TASK-19** — PUT /clients/{idClient}/plan

> Before starting Phase 4: run the clarification gate for each endpoint.
> Write tests first. Update this file with the task list before implementing.

---

## Claude Prompt Templates

### Start a task
```
Read AGENTS.md, CONSTITUTION.md, requirements.md, CLARIFICATION_GATE.md, and plan.md first.
Now implement TASK-14: add ClientUseCase.findAddresses as described in the Active Task section.
Do not implement TASK-15 yet. If anything is unclear, flag it [NEEDS_CLARIFICATION] and stop.
```

### Continue after approval
```
TASK-14 approved. Now implement TASK-15.
Re-read requirements.md §2.3 and CLARIFICATION_GATE.md Q2 before starting.
```

### After a test failure
```
This test is failing: [paste error]
Before guessing, re-read CONSTITUTION.md and check if this pattern is covered there.
Fix the issue. Do not modify the test unless the requirement itself changed.
```
