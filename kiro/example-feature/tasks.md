# Tasks — Expired Link Handling

> **Feature ID:** FEAT-042
> **Linked:** requirements.md · design.md
> **Date:** 2026-04-11
>
> Rule: implement one task at a time. After each task, stop and wait for approval.
> Do not start the next task until the current one is verified.

---

## Task List

### TASK-1 · DB migrations

**Objective:** Add `expires_at` to `short_links` and `resolution_status` to `click_events`.

**Files:**
- `src/main/resources/db/migration/V5__Add_expiry_to_short_links.sql`
- `src/main/resources/db/migration/V6__Add_resolution_status_to_click_events.sql`

**Acceptance:**
```
./mvnw flyway:migrate
→ Both migrations applied successfully, zero errors.
```

**Claude prompt:**
> Write two Flyway migrations:
> 1. `V5__Add_expiry_to_short_links.sql` — add nullable `expires_at TIMESTAMP WITH TIME ZONE` to `short_links`, plus index `idx_short_links_expires_at`.
> 2. `V6__Add_resolution_status_to_click_events.sql` — add `resolution_status VARCHAR(20) NOT NULL DEFAULT 'ACTIVE'` to `click_events`.
> Follow the naming convention in `steering/style.md`.

---

### TASK-2 · Domain model update

**Objective:** Add `expiresAt` field to the `ShortLink` entity and `resolutionStatus` to `ClickEvent`.

**Files:**
- `src/main/java/com/example/link/model/ShortLink.java`
- `src/main/java/com/example/link/model/ClickEvent.java`
- `src/main/java/com/example/link/model/ResolutionStatus.java` _(new enum)_

**Acceptance:**
```
./mvnw compile
→ Zero compilation errors.
```

**Claude prompt:**
> Update `ShortLink` entity: add `expiresAt` field (`OffsetDateTime`, nullable).
> Update `ClickEvent` entity: add `resolutionStatus` field using the `ResolutionStatus` enum.
> Create `ResolutionStatus` enum with values: `ACTIVE`, `EXPIRED`, `NOT_FOUND`.
> Follow naming conventions in `steering/style.md`.

---

### TASK-3 · Link state resolution logic

**Objective:** Implement the three-state resolution logic (ACTIVE / EXPIRED / NOT_FOUND) in the service layer. No controller changes yet.

**Files:**
- `src/main/java/com/example/link/service/LinkResolutionService.java` _(new interface)_
- `src/main/java/com/example/link/service/LinkResolutionServiceImpl.java` _(new class)_
- `src/main/java/com/example/link/exception/LinkExpiredException.java` _(new)_

**Acceptance:**
```
./mvnw test -Dtest=LinkResolutionServiceTest
→ All unit tests pass.
```

**Claude prompt:**
> Create `LinkResolutionService` interface with a single method `resolve(String token)` returning `ResolutionResult` (a sealed interface with three records: `Active(String destinationUrl)`, `Expired`, `NotFound`).
> Implement in `LinkResolutionServiceImpl`:
> - If token is found and `expiresAt` is null or in the future → return `Active`
> - If token is found and `expiresAt` is in the past → return `Expired`
> - If token not found → return `NotFound`
> Clock: use `OffsetDateTime.now(ZoneOffset.UTC)` — do not use system default timezone.
> Write unit tests in `LinkResolutionServiceTest` covering all three states and the null-expiry case.

---

### TASK-4 · Token format validation

**Objective:** Add `@Pattern` validation to reject tokens with invalid characters before any DB access.

**Files:**
- `src/main/java/com/example/link/controller/LinkResolutionController.java`
- `src/main/java/com/example/link/exception/GlobalExceptionHandler.java`

**Acceptance:**
```
./mvnw test -Dtest=LinkResolutionControllerTest#shouldReturn400_whenTokenContainsInvalidCharacters
→ Test passes.
```

**Claude prompt:**
> In `LinkResolutionController`, add `@Pattern(regexp = "[a-zA-Z0-9_\\-]{1,32}")` to the `token` path variable.
> In `GlobalExceptionHandler`, add a handler for `ConstraintViolationException` that returns HTTP 400 with error code `INVALID_TOKEN`.
> The handler must NOT include the destination URL anywhere in the response.
> Write one unit test verifying the 400 response for a token with invalid characters.

---

### TASK-5 · Controller — wire resolution states to HTTP responses

**Objective:** Map `Active` → 302, `Expired` → 410, `NotFound` → 404 in the controller.

**Files:**
- `src/main/java/com/example/link/controller/LinkResolutionController.java`
- `src/main/java/com/example/link/exception/GlobalExceptionHandler.java`

**Acceptance:**
```
./mvnw test -Dtest=LinkResolutionControllerTest
→ All controller unit tests pass.
```

**Claude prompt:**
> Update `LinkResolutionController.resolve()` to call `LinkResolutionService.resolve(token)` and switch on the result:
> - `Active(url)` → `ResponseEntity.status(302).header("Location", url).build()`
> - `Expired` → throw `LinkExpiredException`
> - `NotFound` → throw `ResourceNotFoundException`
> In `GlobalExceptionHandler`, add handler for `LinkExpiredException` → HTTP 410, error code `LINK_EXPIRED`.
> Error responses must NOT include the destination URL.

---

### TASK-6 · Async click event recording

**Objective:** Record a `click_event` with the correct `resolutionStatus` for every request, asynchronously.

**Files:**
- `src/main/java/com/example/link/service/ClickEventService.java`
- `src/main/java/com/example/link/config/AsyncConfig.java` _(new)_

**Acceptance:**
```
./mvnw test -Dtest=ClickEventServiceTest
→ All tests pass.
Verify: click_event row is written in the integration test DB after each resolution.
```

**Claude prompt:**
> Create `ClickEventService` with an `@Async` method `record(String token, ResolutionStatus status)`.
> Configure a dedicated thread pool in `AsyncConfig` (core: 2, max: 10, queue: 500, name: `clickEventExecutor`).
> If the write throws any exception, log at WARN level and swallow — do not propagate to the caller.
> Wire `ClickEventService.record()` into `LinkResolutionServiceImpl` after the resolution decision is made.
> Write unit tests verifying: (1) async call is made for each resolution state, (2) exception in write does not propagate.

---

### TASK-7 · Integration tests (full slice)

**Objective:** Cover all acceptance criteria from `requirements.md` Section 6 with Testcontainers integration tests.

**Files:**
- `src/test/java/com/example/link/controller/LinkResolutionIT.java` _(new)_

**Acceptance:**
```
./mvnw verify
→ All tests pass, coverage ≥ 80%, zero failures.
```

**Claude prompt:**
> Create `LinkResolutionIT` using `@SpringBootTest` + `@Testcontainers` with a real PostgreSQL container.
> Cover every acceptance criterion from `requirements.md` Section 6:
> - Active link → 302 with correct Location header
> - Expired link → 410, no Location header, no destination URL in body
> - Unknown token → 404
> - Invalid token characters → 400, no DB hit (assert click_events count unchanged)
> - Null expires_at → treated as active
> - click_event written for each case with correct resolution_status
> - click_count not incremented for expired/not-found
> Use `@DisplayName` on each test. Follow the `should{Result}_when{Condition}` naming convention.
