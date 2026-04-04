# Tasks — Product API (CRUD)

> **Feature ID:** FEAT-001
> **Linked:** requirements.md · design.md · CONSTITUTION.md
> **Date:** 2025-01-01

---

## Instructions for Claude

```
Before starting any task:
1. Read CONSTITUTION.md (global rules, always apply)
2. Read requirements.md (WHAT to build)
3. Read design.md (HOW to build it)

Implement ONE task at a time.
After each task, wait for my approval before continuing.
Log any unexpected behavior in lessons_learned.md.
Never skip a task or combine tasks without asking first.
```

---

## Phase 1 — Foundation

- [ ] **TASK-01** — Create Maven project with Spring Boot 3.x, add dependencies:
  `spring-boot-starter-web`, `spring-boot-starter-data-jpa`,
  `spring-boot-starter-security`, `spring-boot-starter-validation`,
  `postgresql`, `lombok`, `jjwt`

- [ ] **TASK-02** — Create `application.yml` with DB config, JWT secret placeholder,
  and `api.base-path=/api/v1`

- [ ] **TASK-03** — Create `ApiResponse<T>` DTO (design.md Section 4)

- [ ] **TASK-04** — Create `GlobalExceptionHandler` with all exception mappings
  (design.md Section 7). Return error envelope, never stack traces.

---

## Phase 2 — Domain Layer

- [ ] **TASK-05** — Create `Product` JPA entity (design.md Section 3).
  Include `@PreUpdate` to set `updatedAt` automatically.

- [ ] **TASK-06** — Create `ProductRepository` extending `JpaRepository<Product, Long>`.
  Add method: `Optional<Product> findBySku(String sku)`
  and `Page<Product> findAllByStatus(String status, Pageable pageable)`

- [ ] **TASK-07** — Create `ProductRequest` DTO with Bean Validation annotations
  matching all rules in requirements.md Section 2.6.

- [ ] **TASK-08** — Create `ProductResponse` DTO.
  Add static factory method `ProductResponse.from(Product product)`.

---

## Phase 3 — Service Layer

- [ ] **TASK-09** — Create `ProductService` interface with methods:
  `create`, `findById`, `findAll`, `update`, `deactivate`

- [ ] **TASK-10** — Implement `ProductServiceImpl`:
  - `create`: check SKU uniqueness → throw `ConflictException` if exists → save
  - `findById`: find or throw `ResourceNotFoundException`
  - `findAll`: return paginated ACTIVE products
  - `update`: find or throw 404 → update fields → save
  - `deactivate`: find or throw 404 → set status INACTIVE → save

---

## Phase 4 — Controller Layer

- [ ] **TASK-11** — Create `ProductController` with all 5 endpoints
  (design.md Section 5). Use `@Valid` on all request bodies.
  Return `ResponseEntity<ApiResponse<ProductResponse>>`.

---

## Phase 5 — Security

- [ ] **TASK-12** — Create `JwtUtil` with `generateToken` and `validateToken` methods

- [ ] **TASK-13** — Create `JwtAuthenticationFilter` extending `OncePerRequestFilter`

- [ ] **TASK-14** — Configure `SecurityFilterChain`: require JWT on all routes,
  allow POST `/api/v1/auth/**` without auth,
  ROLE_ADMIN required for POST/PUT/DELETE on `/api/v1/products`

---

## Phase 6 — Tests

- [ ] **TASK-15** — Unit tests for `ProductServiceImpl` (all methods, happy + error paths)

- [ ] **TASK-16** — Integration tests for `ProductController`
  (all endpoints, all acceptance criteria from requirements.md Section 6)

- [ ] **TASK-17** — Run full test suite. Confirm all tests pass. Fix failures.

---

## Claude Prompt Templates

### Start a task
```
Read CONSTITUTION.md, requirements.md, and design.md in specs/active/FEAT-001/.
Now implement TASK-05: Create the Product JPA entity as described in design.md Section 3.
Do not implement TASK-06 yet.
```

### Continue after approval
```
TASK-05 approved. Now implement TASK-06.
Before starting, re-read design.md Section 3 and requirements.md Section 2.6.
```

### After a test failure
```
This test is failing: [paste error]
Check lessons_learned.md first.
Fix the issue, then update lessons_learned.md with the root cause and fix.
```

### Full feature kickoff prompt
```
I want to build a Product CRUD API using Java Spring Boot.
Read all files in specs/active/FEAT-001/ before starting.
Start with TASK-01. Follow CONSTITUTION.md at all times.
After completing each task, summarize what you built and wait for my approval.
```
