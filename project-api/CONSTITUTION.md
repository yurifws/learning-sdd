# Project Constitution
> Global rules Claude ALWAYS follows in this project. Never override these.

---

## Stack

- **Language:** Java 17+
- **Framework:** Spring Boot 3.x
- **Build:** Maven
- **Database:** PostgreSQL (JPA + Hibernate)
- **Auth:** JWT (Bearer token)
- **Tests:** JUnit 5 + Mockito

---

## Architecture Rules

- Follow **layered architecture**: Controller → Service → Repository
- Controllers handle HTTP only — no business logic
- Services contain all business logic — no direct DB calls
- Repositories extend `JpaRepository` — no native SQL unless justified in spec
- DTOs for all request/response bodies — never expose entities directly

---

## Code Standards

- All classes use constructor injection (`@RequiredArgsConstructor`)
- No `@Autowired` on fields
- Use `Optional<T>` from repository — never return `null`
- All public methods in Service must have Javadoc
- Exceptions thrown from Service, handled in `GlobalExceptionHandler`

---

## API Contract Rules

- Base path: `/api/v1`
- All responses wrapped in `ApiResponse<T>` envelope
- Success: HTTP 200/201 with `{ "data": ..., "message": "..." }`
- Error: appropriate HTTP code with `{ "error": "code", "message": "..." }`
- Timestamps always in ISO-8601 format
- IDs always as `Long`

---

## Error Code Standard

| HTTP | Error Code | When |
|------|-----------|------|
| 400 | `VALIDATION_ERROR` | Invalid input fields |
| 401 | `UNAUTHORIZED` | Missing or invalid JWT |
| 403 | `FORBIDDEN` | Valid JWT but no permission |
| 404 | `NOT_FOUND` | Resource does not exist |
| 409 | `CONFLICT` | Duplicate or state conflict |
| 500 | `INTERNAL_ERROR` | Unexpected server error |

---

## Testing Rules

- Every Service method must have a unit test
- Every Controller endpoint must have an integration test (`@SpringBootTest`)
- Test class naming: `[ClassName]Test`
- Use `@DisplayName` on every test method
- Minimum coverage: 80%

---

## Security Rules

- All endpoints require JWT unless annotated `@Public`
- Never log sensitive data (passwords, tokens, CPF, credit cards)
- Always validate and sanitize inputs with Bean Validation (`@Valid`)
- Never expose stack traces in API responses

---

## Enforcement — 3-Tier System

### Always do
- Re-read `CONSTITUTION.md` before starting each task — follow every rule without exception
- Run `./mvnw test` after every task — all tests must pass before marking it done
- Run the unit test for any service method you add or modify
- Update `tasks.md` (check off completed tasks) and `lessons_learned.md` (log any errors)
- Use DTOs for all request/response — never expose JPA entities directly
- Return `ApiResponse<T>` envelope on every endpoint

### Ask before doing
- Adding any new Maven dependency
- Changing the database schema or existing migrations
- Changing any public API contract, endpoint path, or response shape
- Modifying `GlobalExceptionHandler` or shared base classes
- Refactoring code outside the scope of the current task

### Never do
- Put business logic in a Controller or Repository
- Return `null` from a public Service method — use `Optional<T>`
- Swallow exceptions silently — always rethrow typed exceptions
- Log passwords, tokens, CPF, credit cards, or any PII
- Expose stack traces in API responses
- Bypass or weaken authentication/authorization logic
- Commit secrets, API keys, or credentials anywhere in the codebase
- Skip failing tests to make a PR pass
