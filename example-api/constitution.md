# constitution.md — Project Rules & Conventions

> These rules are NON-NEGOTIABLE. Always follow them in every file you create or modify.

---

## Architecture

This project uses **Hexagonal Architecture (Ports & Adapters)**.

- `adapters/input/controller` → Handles HTTP (REST). No business logic here.
- `adapters/input/grpc` → Handles gRPC. No business logic here.
- `adapters/output/persistence` → JPA repositories and persistence services.
- `application/ports/in` → Input port interfaces (what the outside world calls).
- `application/ports/out` → Output port interfaces (what the app needs from outside).
- `application/service` → Use cases. All business logic lives here.
- `domain` → Plain Java POJOs. No JPA, no Spring annotations.
- `config` → Spring Boot configuration classes.

### Golden Rules
- Controllers ONLY handle HTTP (parse request, call PortIn, return response).
- Use cases implement `PortIn` and only call `PortOut` interfaces — never repositories directly.
- Persistence services implement `PortOut` and call repositories.
- Domain objects are pure Java — zero framework annotations.
- Never call a repository from a controller or use case.

---

## Naming Conventions

| Layer | Pattern | Example |
|---|---|---|
| Controller | `{Domain}Controller` | `ClientController` |
| OpenAPI Interface | `{Domain}OpenApi` | `ClientOpenApi` |
| Input Port | `{Domain}PortIn` | `ClientPortIn` |
| Output Port | `{Domain}PersistencePortOut` | `ClientPersistencePortOut` |
| Use Case | `{Domain}UseCase` | `ClientUseCase` |
| Persistence Service | `{Domain}PersistenceService` | `ClientPersistenceService` |
| Repository | `{Entity}Repository` | `ClientRepository` |
| Domain Model | `{Domain}Domain` | `ClientDomain` |
| Response Model | `{Domain}ResponseModel` | `ClientSummaryResponseModel` |
| Query Result DTO | `{Feature}QueryResultDTO` | `ClientSummaryQueryResultDTO` |
| Controller Mapper | `{Domain}ControllerMapper` | `ClientControllerMapper` |
| Persistence Mapper | `{Domain}PersistenceMapper` | `ClientPersistenceMapper` |

---

## Mappers

- Always use **MapStruct**. Never map fields manually.
- All mappers extend `BaseMapper`.
- Use `INSTANCE = Mappers.getMapper(...)` singleton pattern.
- Use `@Mapping` with `qualifiedByName` for custom conversions (e.g., `toBoolean`).

```java
@Mapper
public interface ClientControllerMapper extends BaseMapper {
    ClientControllerMapper INSTANCE = Mappers.getMapper(ClientControllerMapper.class);
    ClientSummaryResponseModel toResponse(ClientDomain domain);
}
```

---

## Security

- Every endpoint in the OpenApi interface **must** have `@PreAuthorize`.
- Roles must be defined as constants in the `Authorities` abstract class.
- Follow the hierarchy: specific role → group role → ADMIN.

```java
public static final String ROLE_CLIENT_QUERY_SUMMARY
    = "hasAnyAuthority('ROLE_CLIENT_QUERY_SUMMARY', " + ROLE_CLIENT_QUERY + ")";
```

---

## Logging

- `log.info()` → on every request received (controller) and success (usecase, persistence).
- `log.warn()` → when an entity is not found.
- `log.error()` → unexpected exceptions.
- Always include the ID or relevant identifier in the log message.

```java
log.info("Request received to find client summary: {}", idClient);
log.warn("Client not found in database: {}", idClient);
```

---

## Error Handling

- Use `NotFoundException` when an entity is not found in persistence.
- Always re-throw typed exceptions — never swallow errors silently.
- Catch `NotFoundException` explicitly in persistence services and rethrow with a clear message.

---

## Response Models

- Use Java **records** for response models.
- Annotate all fields with `@Schema` (description + example).
- Use `@JsonFormat(pattern = "yyyy-MM-dd")` for `LocalDate` fields.
- Nested objects should be inner records inside the parent record.

---

## Entities

- All entities extend `BaseEntity` (which provides `@Id Long id`).
- Use `@AttributeOverride` to map the correct column name for the ID.
- Always implement `equals()` and `hashCode()` based on `getId()`.
- Use `@Table(name = "T_TABLENAME")` and `@Column(name = "COLUMNNAME")`.

---

## OpenAPI

- Every OpenApi interface must be annotated with `@Tag`.
- Every method must have `@Operation` (summary + description).
- Every response code must have `@ApiResponse` with content schema.
- Include `@DefaultErrorApiResponses` for standard error codes.
- Use `@Parameter` with description, required, and example on every path/query param.

---

## Claude Behavior Rules

- Always follow the hexagonal layers — never skip a layer.
- Ask before creating files not in the current task.
- Prefer editing existing files over creating new ones.
- If a task is ambiguous, ask for clarification before writing code.
- Generate all 6 layers for a new endpoint: Controller → OpenApi → PortIn → UseCase → PortOut → PersistenceService.
