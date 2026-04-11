# PROJECT_TECHSTACK.md — Task Management API

> This is a filled-in example. See the blank template at `agent-os/PROJECT_TECHSTACK.md`.

---

## Language & Runtime

| Concern | Decision | Version |
|---|---|---|
| Language | Java | 21 |
| Framework | Spring Boot | 3.3 |
| Build tool | Maven | 3.9 |

---

## Key Libraries

| Role | Library | Version | Notes |
|---|---|---|---|
| Database access | Spring Data JPA + Hibernate | 6.x (via Boot) | Repositories only in persistence layer |
| Database migrations | Flyway | 10.x (via Boot) | All schema changes via migration scripts |
| Auth | Spring Security + jjwt | 0.12.x | RS256, public/private key pair |
| API docs | springdoc-openapi | 2.x | Swagger UI at `/swagger-ui.html` |
| Mapping | MapStruct | 1.5.x | DTO ↔ domain mapping, no manual builders |
| Testing | JUnit 5 + Testcontainers | 3.x (via Boot) | Integration tests hit a real PostgreSQL container |
| Validation | Jakarta Validation (Hibernate Validator) | via Boot | Bean validation on request DTOs only |

---

## Architecture Pattern

Hexagonal Architecture (Ports & Adapters).

```
adapters/input/controller     ← HTTP only, no business logic
application/ports/in          ← input port interfaces
application/service           ← use cases, all business logic
application/ports/out         ← output port interfaces
adapters/output/persistence   ← JPA repositories + persistence services
domain                        ← pure Java, zero framework annotations
```

Flow: `Controller → PortIn → UseCase → PortOut → PersistenceService → JpaRepository`

---

## Banned / Off-limits

- No Lombok — use Java records for DTOs, plain classes for domain objects
- No field injection (`@Autowired` on fields) — constructor injection only
- No business logic in controllers or persistence services
- No JPA annotations on domain objects — only on `@Entity` classes in the persistence layer
- No `Optional.get()` without `isPresent()` check — use `orElseThrow` instead
- No `System.out.println` — use SLF4J (`log.info`, `log.error`)

---

## Infrastructure

| Concern | Decision |
|---|---|
| Database | PostgreSQL 16 |
| Auth mechanism | JWT, RS256 (self-signed key pair for dev, injected via env in prod) |
| Deployment target | Docker + Railway |
| CI | GitHub Actions (`./mvnw verify` on every PR) |
| Containerization | Dockerfile with multi-stage build (build → runtime on Eclipse Temurin 21 slim) |