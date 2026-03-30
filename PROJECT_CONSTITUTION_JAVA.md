# PROJECT CONSTITUTION — JAVA BACKEND

**Project:** [Your Project Name]
**Version:** 1.0
**Last updated:** [Date]

---

## Article 1 — Architectural principles

- **Layer-first:** Business logic lives in the service layer only. Controllers are thin (routing + validation only). Repositories are thin (data access only). No business logic leaks into either.
- **Interface-first:** Every service must have an interface before its implementation. Code against interfaces, never concrete classes.
- **Locked stack:** [e.g. Java 21, Spring Boot 3.x, Hibernate 6.x, PostgreSQL 15] — no upgrades without team sign-off.
- **No premature abstraction:** Don't build a custom framework around something Spring already provides.
- **Simplicity over cleverness:** If a method needs a comment to explain what it does, simplify it first.
- **Immutability by default:** Prefer `final` fields, immutable DTOs, and records where possible.

---

## Article 2 — Coding standards

**Naming**
- Classes and interfaces: `PascalCase`
- Methods and variables: `camelCase`
- Constants: `UPPER_SNAKE_CASE`
- Packages: `lowercase.with.dots` (e.g. `com.yourcompany.projectname.users`)
- Database columns: `snake_case`

**Package structure**
```
com.yourcompany.projectname
  ├── controller      ← HTTP layer only, no business logic
  ├── service         ← all business logic lives here
  │     └── impl      ← service implementations
  ├── repository      ← data access only (Spring Data / JPA)
  ├── domain          ← entities and domain models
  ├── dto             ← request/response objects (records preferred)
  ├── config          ← Spring configuration classes
  ├── exception       ← custom exception classes
  └── util            ← stateless utility classes only
```

**Error handling**
- Always use typed custom exceptions (e.g. `UserNotFoundException extends RuntimeException`)
- Never throw raw `RuntimeException` or `Exception` with a plain string
- Every controller must have a `@ControllerAdvice` handler — no unhandled exceptions reach the client
- No swallowed exceptions — never catch and do nothing (`catch (Exception e) {}`)

**Code style**
- Max method length: 20 lines — extract if longer
- Max class length: 200 lines — split if longer
- No `null` returns from public methods — use `Optional<T>` instead
- No `System.out.println` — use SLF4J (`private static final Logger log = LoggerFactory.getLogger(...)`)

**Dependencies (Maven/Gradle)**

| Allowed | Forbidden |
|---|---|
| `spring-boot-starter-*` (official) | Any lib not in Maven Central |
| `hibernate-core` | `Apache Commons` (use Java standard library) |
| `lombok` | Any lib with fewer than 1000 GitHub stars |
| `mapstruct` | Reflection-heavy libs without justification |
| `testcontainers` | In-memory DBs for integration tests (use real DB via Testcontainers) |
| [add your approved libs] | Any lib not actively maintained (last release > 2 years) |

---

## Article 3 — Testing rules (non-negotiable)

- **Test-first:** Write the test before the implementation for any new feature or bug fix
- **Coverage floor:** Minimum [80]% line coverage — PRs below this are rejected
- **No `@Disabled`:** Disabling a test in committed code is forbidden — fix it or delete it
- **Unit tests:** Every service method gets a unit test with Mockito — no Spring context loaded
- **Integration tests:** Every controller endpoint gets an integration test with `@SpringBootTest` and Testcontainers
- **Contract-first for APIs:** Define request/response DTOs and OpenAPI spec before writing any controller
- **Test naming:** `methodName_givenCondition_expectedResult` (e.g. `findUser_givenInvalidId_throwsNotFoundException`)
- **Test file location:** Every `FooService.java` must have a `FooServiceTest.java` in the mirrored `/src/test/java` path

**Test stack**
- Unit tests: JUnit 5 + Mockito
- Integration tests: `@SpringBootTest` + Testcontainers + RestAssured or MockMvc
- Assertions: AssertJ (never plain JUnit `assertEquals`)

---

## Article 4 — The 3-tier enforcement system

### Always do (no questions asked)
- Run `./mvnw verify` (or `./gradlew check`) before every commit — all tests must pass
- Keep PRs small — one feature or fix per PR, max ~400 lines changed
- Follow the package structure and naming conventions above
- Write a unit test for every new service method
- Use `@Slf4j` (Lombok) or manual SLF4J for all logging

### Ask a human first
- Adding any new Maven/Gradle dependency
- Changing the database schema (migrations must be reviewed)
- Modifying a shared interface or DTO used by multiple services
- Adding a new Spring configuration or changing application properties
- Any performance optimization that adds significant complexity
- Introducing a new design pattern not already used in the project

### Never do (hard red lines)
- Commit secrets, passwords, API keys, or credentials anywhere in the codebase
- Use raw `Object` type where a generic or typed class should be used
- Catch and swallow exceptions silently
- Use `System.out.println` for logging
- Write business logic inside a `@Controller` or `@Repository` class
- Push directly to `main` or `master`
- Disable or skip failing tests to make a PR pass
- Use a forbidden or unapproved dependency

---

## Article 5 — Architecture gates

Before writing any code, answer these questions. If you can't answer them clearly, stop and ask.

**Simplicity gate**
> "Is this complexity actually justified by the spec I was given?"

**Layer gate**
> "Am I putting this logic in the right layer — controller, service, or repository?"

**Abstraction gate**
> "Am I building a custom solution because I need to, or does Spring already provide this?"

**Dependency gate**
> "Is this library on the allowed list? Could I implement this with the standard Java library instead?"

**Test gate**
> "Do I have a test written for this before I build it?"

**Null gate**
> "Am I returning or accepting `null` anywhere? Should this be `Optional<T>` instead?"

---

## Article 6 — Database & migrations

- All schema changes go through versioned migration files (Flyway or Liquibase)
- Migration filenames: `V{version}__{description}.sql` (e.g. `V002__add_user_email_index.sql`)
- Never edit an already-applied migration — always create a new one
- Every new table must have: `id`, `created_at`, `updated_at` columns
- Foreign keys must be explicit and indexed
- No raw SQL in service or repository classes — use JPA/JPQL or a named query

---

## Article 7 — Glossary

| Term | Definition |
|---|---|
| Service | A `@Service` class containing all business logic for a domain area |
| DTO | Data Transfer Object — a record or class used to move data between layers |
| Entity | A `@Entity` JPA class mapped directly to a database table |
| Repository | A `@Repository` interface extending Spring Data for data access |
| [Term] | [What it means in this project] |

---

*This constitution applies to all contributors — human and AI alike. When in doubt, refer here first.*
