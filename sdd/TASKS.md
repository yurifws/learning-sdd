# tasks.md — [Project Name] / [Feature Name]

> **Spec reference:** `specs/active/[FEAT-XXX]/requirements.md` and `design.md`
> **Constitution:** `CONSTITUTION.md`
> **Status:** 🔴 In Progress | **Date:** YYYY-MM-DD
>
> AI agent: work through this file **top to bottom, one task at a time.**
> Check off each task only when the verify command passes.

---

## Ground Rules (Read Before Starting)

- Touch **only the files listed** in each task — nothing else
- Each task has **one objective** — no side-quests, no unrequested refactoring
- If a task feels too big or ambiguous, **stop and ask** before proceeding
- `[p]` = safe to parallelize — only if the file boundaries don't overlap
- Mark a task `✅` only when the verify step passes
- If a verify command fails for a reason unrelated to your task, **flag it** — do not silently fix it

---

## Phase 1 — Project Setup

- [ ] **T-01** · Initialize project structure
  **Files:** `[e.g., package.json / pom.xml / go.mod / pyproject.toml]`, `[config file]`
  **Objective:** Scaffold the project with required dependencies and base configuration
  **Verify:** `[YOUR_BUILD_COMMAND]` completes with no errors
  **Ref:** Constitution § Tech Stack

- [ ] **T-02** · Configure environment variables
  **Files:** `[config file]`, `.env.example`
  **Objective:** Define all required env vars and provide a documented example file
  **Verify:** Application starts without missing configuration errors
  **Ref:** Constitution § Security Gate

---

## Phase 2 — Data Foundation

- [ ] **T-03** · Create `[Entity]` schema / data model
  **Files:** `[e.g., migrations/V1__create_entity.sql / models/entity.py / schema.prisma]`
  **Objective:** Define the data structure with all fields, constraints, and indexes
  **Verify:** `[YOUR_MIGRATION_COMMAND]` runs cleanly; schema matches the design doc
  **Ref:** `design.md` § Data Models

- [ ] **T-04** · Create `[Entity]` data access layer
  **Files:** `[e.g., repository/entity_repo.go / repositories/entity.py / src/.../EntityRepository.java]`
  **Objective:** Implement read/write operations for `[Entity]`
  **Verify:** `[YOUR_TEST_COMMAND] [DataLayerTestFile]` passes
  **Ref:** `design.md` § Data Models

---

## Phase 3 — API Contracts

- [ ] **T-05** · Define request and response shapes for `[endpoint]`
  **Files:** `[e.g., dto/create_entity.go / schemas/entity.py / dto/EntityRequest.java]`
  **Objective:** Define the input/output data structures matching the API contract in `design.md`
  **Verify:** Types compile / schema validates with no errors
  **Ref:** `design.md` § API Endpoints

- [ ] **T-06** · Create `[Feature]` route/controller with stubs
  **Files:** `[e.g., handlers/entity.go / routers/entity.py / controllers/EntityController.java]`
  **Objective:** Register the routes and return stub responses (e.g., 501 Not Implemented)
  **Verify:** `[YOUR_HTTP_CALL] [YOUR_ENDPOINT]` returns the stub status code
  **Ref:** `design.md` § API Endpoints

---

## Phase 4 — Core Logic

- [ ] **T-07** · Implement `[Feature]` — happy path
  **Files:** `[e.g., services/entity_service.go / services/entity.py / service/EntityService.java]`
  **Objective:** Implement the main business logic for the success scenario
  **Verify:** `[YOUR_TEST_COMMAND] [HappyPathTestFile]` passes
  **Ref:** `requirements.md` — [EARS clause ID for the happy path]

- [ ] **T-08** · Implement `[Feature]` — error cases
  **Files:** `[same service file as T-07]`
  **Objective:** Handle all failure scenarios defined in the EARS requirements
  **Verify:** `[YOUR_TEST_COMMAND] [ErrorCaseTestFile]` passes
  **Ref:** `requirements.md` — [EARS clause IDs for error paths]

- [ ] **T-09** · Wire service into route/controller
  **Files:** `[handler / controller file from T-06]`
  **Objective:** Replace stubs with real service calls; map results to correct HTTP responses
  **Verify:** `[YOUR_INTEGRATION_TEST_COMMAND]` passes end-to-end
  **Ref:** `design.md` § API Endpoints

---

## Phase 5 — Security

- [ ] **T-10** · Add input validation
  **Files:** `[request schema / DTO / validator file]`
  **Objective:** Validate all required fields and reject malformed requests before they reach business logic
  **Verify:** `[YOUR_HTTP_CALL]` with an empty or invalid body returns the correct 4xx status and error code
  **Ref:** `requirements.md` — [validation EARS clause ID]

- [ ] **T-11** · Enforce authentication on protected routes
  **Files:** `[middleware / security config / auth guard file]`
  **Objective:** Require a valid auth token for all protected endpoints
  **Verify:** Request without a token returns 401; request with an invalid token returns 401
  **Ref:** `requirements.md` — [auth EARS clause ID]

---

## Phase 6 — Tests & Verification

- [ ] **T-12 [p]** · Write unit tests for the service layer
  **Files:** `[test file mirroring the service file from T-07/T-08]`
  **Objective:** Cover all EARS success criteria and error paths with isolated unit tests
  **Verify:** `[YOUR_TEST_COMMAND] [UnitTestFile]` — all pass
  **Ref:** `requirements.md` § Success Criteria

- [ ] **T-13 [p]** · Write integration tests for the route/controller
  **Files:** `[test file mirroring the handler/controller file]`
  **Objective:** Test the full request/response cycle including auth, validation, and error responses
  **Verify:** `[YOUR_INTEGRATION_TEST_COMMAND]` — all pass
  **Ref:** `requirements.md` § Error Handling

- [ ] **T-14** · Confirm no sensitive data in logs or responses
  **Files:** `[service file, handler file]`
  **Objective:** Verify that passwords, tokens, and PII are never logged or returned in error responses
  **Verify:** Run the feature end-to-end; grep logs for sensitive field names — none found
  **Ref:** Constitution § Security Gate

---

## Human Review Checklist

> Complete this before handing the file to the AI agent.

- [ ] Every EARS requirement from `requirements.md` maps to at least one task
- [ ] No task touches more than 1–2 files
- [ ] No task has more than one objective
- [ ] Every task has a specific, runnable verify command — not just "run the tests"
- [ ] Tasks are ordered correctly — no task depends on one listed after it
- [ ] `[p]` tasks genuinely have no overlapping files or shared state
- [ ] No mega-tasks hiding as a single line
- [ ] Reviewed by: **[Name]** on **YYYY-MM-DD**

> **Approved. Agent may begin at T-01.**
