# steering/techstack.md

> This file is read by the AI agent before every session.
> It defines locked stack decisions, version pins, and banned patterns.
> Do not change without a team decision — treat it as a contract.

---

## Locked Stack

| Layer | Technology | Version |
|---|---|---|
| Language | Java | 21 (LTS) |
| Framework | Spring Boot | 3.3.x |
| Build | Maven | 3.9.x |
| Database | PostgreSQL | 16.x |
| ORM | Spring Data JPA + Hibernate | (managed by Boot) |
| Migrations | Flyway | 10.x |
| Auth | Spring Security + JWT (RS256) | — |
| Tests | JUnit 5 + Testcontainers | — |
| API Docs | SpringDoc OpenAPI 3 | 2.x |

---

## Version Pins

Always resolve to these exact versions. Do not upgrade without updating this file first.

```xml
<java.version>21</java.version>
<spring-boot.version>3.3.4</spring-boot.version>
<testcontainers.version>1.19.8</testcontainers.version>
<springdoc.version>2.6.0</springdoc.version>
<flyway.version>10.15.0</flyway.version>
```

---

## Banned Patterns

These are hard stops. If you are about to use any of these, stop and flag it.

| Pattern | Why it is banned |
|---|---|
| `System.out.println` | Use `Slf4j` logger only |
| `@Autowired` on fields | Use constructor injection — testability |
| `Optional.get()` without `isPresent()` | Causes silent NPEs |
| Raw `EntityManager` queries | Use Spring Data JPA or `@Query` |
| Hardcoded credentials | Never in source — use environment variables |
| `Thread.sleep()` in tests | Use `Awaitility` |
| `*.env` files committed to git | Blocked by Never hook — see `hooks/HOOKS.md` |

---

## Infrastructure Notes

- Local dev: Docker Compose (`docker-compose.yml` at project root)
- CI: GitHub Actions — `.github/workflows/ci.yml`
- Secrets: loaded from environment, never from `application.properties`
- DB migrations run automatically on startup via Flyway
