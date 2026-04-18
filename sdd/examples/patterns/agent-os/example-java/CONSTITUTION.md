# CONSTITUTION.md — Task Management API Rules

> These rules are NON-NEGOTIABLE. Apply them in every file you create or modify.
> Read this file before touching any code, spec, or task.

---

## Architecture

This project uses **Hexagonal Architecture (Ports & Adapters)**.

```
adapters/input/controller   ← HTTP only. No business logic. No repository calls.
application/ports/in        ← Input port interfaces (what the outside world calls).
application/service         ← Use cases. All business logic lives here.
application/ports/out       ← Output port interfaces (what the app needs from outside).
adapters/output/persistence ← Persistence services + JPA repositories.
domain                      ← Pure Java classes. No Spring, no JPA.
```

**Flow:** Controller → PortIn → UseCase → PortOut → PersistenceService → JpaRepository

### Golden Rules
- Controllers ONLY parse requests, call a PortIn, and return a response.
- Use cases implement PortIn and only call PortOut interfaces — never JPA repositories directly.
- Persistence services implement PortOut and call JpaRepositories.
- Domain objects are pure Java — zero framework annotations.
- Status transition logic lives in the domain class, not in the controller or service.

---

## Naming Conventions

| Layer | Pattern | Example |
|---|---|---|
| Controller | `{Domain}Controller` | `TaskController` |
| OpenAPI Interface | `{Domain}OpenApi` | `TaskOpenApi` |
| Input Port | `{Domain}PortIn` | `TaskPortIn` |
| Output Port | `{Domain}PersistencePortOut` | `TaskPersistencePortOut` |
| Use Case | `{Domain}UseCase` | `TaskUseCase` |
| Persistence Service | `{Domain}PersistenceService` | `TaskPersistenceService` |
| JPA Repository | `{Entity}Repository` | `TaskRepository` |
| Domain Model | `{Domain}Domain` | `TaskDomain` |
| Request DTO | `{Action}{Domain}Request` | `CreateTaskRequest` |
| Response DTO | `{Domain}Response` | `TaskResponse` |
| Persistence Mapper | `{Domain}PersistenceMapper` | `TaskPersistenceMapper` |
| Controller Mapper | `{Domain}ControllerMapper` | `TaskControllerMapper` |

---

## No Lombok

Use Java records for DTOs. Use plain classes for domain objects.

```java
// DTO — use a record
public record CreateTaskRequest(
    @NotBlank String title,
    String description
) {}

// Domain — use a plain class
public class TaskDomain {
    private Long id;
    private String title;
    private TaskStatus status;
    // getters/setters
}
```

---

## Mapping

- Use **MapStruct** for all DTO ↔ domain and domain ↔ entity conversions.
- Never map fields manually.
- All mappers implement a shared `BaseMapper` interface.

---

## Security

- Every endpoint in the OpenApi interface **must** have `@PreAuthorize`.
- Roles: `ROLE_ADMIN` (create, update, delete), `ROLE_USER` (read, assign own tasks).
- JWT uses RS256 — public/private key pair, never symmetric.
- Never log tokens, passwords, or any PII.

---

## Status Transitions

Valid transitions (enforced in `TaskDomain`):
```
TODO        → IN_PROGRESS
IN_PROGRESS → DONE
IN_PROGRESS → TODO          (unblock)
```

Invalid transitions must throw `InvalidStatusTransitionException` at the domain layer.
Never put transition guards in a controller.

---

## Error Handling

| Exception | HTTP | Code |
|---|---|---|
| `ResourceNotFoundException` | 404 | NOT_FOUND |
| `ConflictException` | 409 | CONFLICT |
| `InvalidStatusTransitionException` | 422 | INVALID_TRANSITION |
| `MethodArgumentNotValidException` | 400 | VALIDATION_ERROR |
| `AccessDeniedException` | 403 | FORBIDDEN |
| Fallback `Exception` | 500 | INTERNAL_ERROR |

Handle all exceptions in `GlobalExceptionHandler`. Never swallow errors silently.

---

## Logging

- `log.info()` — every request received (controller), every successful operation (use case).
- `log.warn()` — entity not found, invalid transition attempted.
- `log.error()` — unexpected exceptions.
- Always include the task ID or relevant identifier.
- Never log: passwords, JWT tokens, user emails, or any PII.

---

## Claude Behavior Rules

### Always do
- Re-read this constitution at the start of every session.
- Update the spec before implementing: requirements.md → design.md → tasks.md → code.
- Commit in order: spec update → plan update → implementation.
- Write tests BEFORE implementation — confirm they fail first (see `TEST_FIRST_GATE.md`).
- Follow all hexagonal layers — never skip a layer.
- Use MapStruct for all mappings — no manual builders or setters.
- Run `./mvnw test` after every task — all tests must pass before marking done.

### Ask before doing
- Creating a file not listed in the current task's acceptance criteria.
- Adding a Maven dependency to `pom.xml`.
- Changing the database schema or migration scripts.
- Changing a public port interface or API contract.
- Modifying `BaseMapper`, `BaseEntity`, or any shared base class.

### Never do
- Call a JPA repository from a controller or use case.
- Put business logic (including status transitions) in a controller or persistence service.
- Add JPA annotations to domain classes.
- Map fields manually — always use MapStruct.
- Swallow exceptions silently.
- Create an endpoint without `@PreAuthorize`.
- Log tokens, passwords, or PII.
- Use `@Autowired` on fields — use constructor injection only.
- Use `Optional.get()` without guarding — use `orElseThrow`.
