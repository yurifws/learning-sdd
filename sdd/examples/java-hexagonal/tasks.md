# tasks.md — Development Tasks

> Ordered task list for the Client API project.
> One task at a time. Stop after each task and wait for approval.
> See `task.md` for the CURRENT active task detail (acceptance criteria, files to touch).

---

## Instructions for Claude

Read `CLAUDE.md` before starting. Then:
1. Find the first unchecked task in the current phase.
2. Read `task.md` — it has the acceptance criteria and exact files to touch for the active task.
3. After each task: run `./mvnw test`, check done criteria, update `plan.md`, wait for approval.

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
Read CLAUDE.md, constitution.md, requirements.md, CLARIFICATION_GATE.md, and plan.md first.
Now implement TASK-14: add ClientUseCase.findAddresses as described in task.md.
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
Before guessing, re-read constitution.md and check if this pattern is covered there.
Fix the issue. Do not modify the test unless the requirement itself changed.
```
