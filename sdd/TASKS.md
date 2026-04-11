# ✅ tasks.md — `[Project Name]`

> **Note:** This template uses Java/Maven commands. Adapt `mvn`/`mvnw` to your build tool if using a different stack.  
> **Plan reference:** `design.md` (or `SDD_REST_API_TEMPLATE.md`) / `CONSTITUTION.md`  
> **Status:** 🔴 In Progress | **Date:** YYYY-MM-DD  
> AI agent: work through this file **top to bottom, one task at a time.**  
> Check off each box only when the verify command passes.

---

## 🔒 Ground Rules (Read Before Starting)

- Touch **only the files listed** in each task — nothing else
- Each task has **one objective** — no side-quests, no refactoring unrelated code
- If a task feels too big, **stop and ask** before proceeding
- `[p]` = safe to parallelize (only if file boundaries don't overlap)
- Mark a task `✅` only when the verify step confirms it works

---

## Phase 1 — Project Setup

- [ ] **T-01** · Initialize project structure  
  **Files:** `pom.xml`, `src/main/resources/application.yml`  
  **Objective:** Scaffold the project with required dependencies and base config  
  **Verify:** `mvn clean install` passes with no errors  
  **Ref:** Constitution § Tech Stack

- [ ] **T-02** · Configure environment variables  
  **Files:** `application.yml`, `.env.example`  
  **Objective:** Define all required env vars (DB url, JWT secret, etc.)  
  **Verify:** App starts without missing property errors  
  **Ref:** Constitution § Security Gate

---

## Phase 2 — Data Foundation

- [ ] **T-03** · Create `[Entity]` database schema  
  **Files:** `src/main/resources/db/migration/V1__create_[entity]_table.sql`  
  **Objective:** Define table with all fields, constraints, and indexes  
  **Verify:** Migration runs cleanly via `flyway:migrate`  
  **Ref:** SDD § Data Models

- [ ] **T-04** · Create `[Entity]` JPA entity class  
  **Files:** `src/main/java/.../model/[Entity].java`  
  **Objective:** Map entity to DB table with correct annotations  
  **Verify:** App starts and Hibernate validates schema  
  **Ref:** SDD § Data Models

- [ ] **T-05** · Create `[Entity]` repository interface  
  **Files:** `src/main/java/.../repository/[Entity]Repository.java`  
  **Objective:** Extend JpaRepository, add any custom query methods  
  **Verify:** Repository bean loads without errors  
  **Ref:** SDD § Data Models

---

## Phase 3 — API Contracts

- [ ] **T-06** · Create request/response DTOs for `[endpoint]`  
  **Files:** `src/main/java/.../dto/[Feature]Request.java`, `[Feature]Response.java`  
  **Objective:** Define input/output shapes matching the API contract  
  **Verify:** Classes compile with correct validation annotations  
  **Ref:** SDD § API Endpoints

- [ ] **T-07** · Create `[Feature]Controller` with endpoint stubs  
  **Files:** `src/main/java/.../controller/[Feature]Controller.java`  
  **Objective:** Define routes, HTTP methods, and return 501 stubs  
  **Verify:** `GET /api/v1/[resource]` returns HTTP 501  
  **Ref:** SDD § API Endpoints

---

## Phase 4 — Core Logic

- [ ] **T-08** · Implement `[Feature]Service` — happy path  
  **Files:** `src/main/java/.../service/[Feature]Service.java`  
  **Objective:** Implement the main business logic for [feature]  
  **Verify:** Unit test `[Feature]ServiceTest#shouldSucceedWhen...` passes  
  **Ref:** SDD § Success Criteria (EARS)

- [ ] **T-09** · Implement `[Feature]Service` — error cases  
  **Files:** `src/main/java/.../service/[Feature]Service.java`  
  **Objective:** Handle all failure scenarios defined in EARS criteria  
  **Verify:** Unit tests for each failure case pass  
  **Ref:** SDD § Success Criteria (EARS)

- [ ] **T-10** · Wire service into controller  
  **Files:** `src/main/java/.../controller/[Feature]Controller.java`  
  **Objective:** Replace stubs with real service calls, map to correct HTTP responses  
  **Verify:** Integration test `[Feature]ControllerTest` passes  
  **Ref:** SDD § API Endpoints

---

## Phase 5 — Security

- [ ] **T-11** · Add input validation to request DTOs  
  **Files:** `src/main/java/.../dto/[Feature]Request.java`  
  **Objective:** Annotate all fields with `@NotNull`, `@Email`, `@Size`, etc.  
  **Verify:** `POST /api/v1/[resource]` with empty body returns HTTP 400  
  **Ref:** Constitution § Security Gate

- [ ] **T-12** · Secure endpoint with auth filter  
  **Files:** `src/main/java/.../security/SecurityConfig.java`  
  **Objective:** Require valid JWT for protected routes  
  **Verify:** Request without token returns HTTP 401  
  **Ref:** SDD § Security

---

## Phase 6 — Tests & Polish

- [ ] **T-13 [p]** · Write unit tests for service layer  
  **Files:** `src/test/java/.../service/[Feature]ServiceTest.java`  
  **Objective:** Cover all EARS success criteria and failure cases  
  **Verify:** `mvn test -pl service` passes  
  **Ref:** SDD § Success Criteria

- [ ] **T-14 [p]** · Write integration tests for controller  
  **Files:** `src/test/java/.../controller/[Feature]ControllerTest.java`  
  **Objective:** Test full request/response cycle including error responses  
  **Verify:** All tests green  
  **Ref:** SDD § Error Handling

- [ ] **T-15** · Verify no sensitive data in logs  
  **Files:** `src/main/java/.../service/[Feature]Service.java`  
  **Objective:** Confirm passwords, tokens, and PII are never logged  
  **Verify:** Run feature, grep logs for sensitive fields — none found  
  **Ref:** Constitution § Security Gate

---

## 🧑‍💻 Human Review Checklist

> Complete before handing this file to the AI agent.

- [ ] Every requirement from the SDD has at least one task
- [ ] No task touches more than 1–2 files
- [ ] No task has more than one objective
- [ ] Every task has a verify command
- [ ] Order is correct (no task depends on one that comes after it)
- [ ] `[p]` tasks truly have no overlapping files or shared state
- [ ] No mega-tasks hiding as a single line
- [ ] Reviewed by: **[Name]** on **YYYY-MM-DD**

> 🔒 **Approved. Agent may begin at T-01.**
