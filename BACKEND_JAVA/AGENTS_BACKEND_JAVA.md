# AGENTS.md — AI Onboarding Packet
# Java Spring Boot — Hexagonal Architecture

> This file is written FOR the AI agent, not for humans.
> Read this entirely before writing or modifying any code.

---

## 1. Project Overview

**Package:** `tech.example.api.[domain]`
**Purpose:** REST API for managing [domain] data.
**Architecture:** Hexagonal Architecture (Ports & Adapters)
**Language:** Java 17+
**Framework:** Spring Boot

---

## 2. Commands

| Task              | Command                        |
|-------------------|--------------------------------|
| Build project     | `./mvnw clean install`         |
| Run application   | `./mvnw spring-boot:run`       |
| Run tests         | `./mvnw test`                  |
| Run single test   | `./mvnw test -Dtest=ClassName` |
| Package JAR       | `./mvnw clean package`         |

> Always use the Maven wrapper (`./mvnw`). Never use a globally installed `mvn`.

---

## 3. Project Map

```
tech.example.api.[domain]
├── adapters
│   ├── input
│   │   ├── controller           → @RestController — HTTP only, no business logic
│   │   │   ├── constant
│   │   │   ├── dto
│   │   │   ├── enums
│   │   │   ├── mapper           → MapStruct — controller layer mappers
│   │   │   ├── openapi          → @RequestMapping interfaces with @PreAuthorize
│   │   │   │   └── apiresponses
│   │   │   └── security         → Authorities constants
│   │   └── grpc                 → gRPC adapter (optional)
│   │       └── mapper
│   └── output
│       └── persistence
│           ├── dto              → Query result projection interfaces
│           ├── entity           → @Entity classes (extend BaseEntity)
│           ├── exceptions       → NotFoundException and typed exceptions
│           ├── mapper           → MapStruct — persistence layer mappers
│           ├── repository       → Spring Data JPA repositories
│           └── service          → implements PortOut — calls repositories
├── application
│   ├── ports
│   │   ├── in                   → PortIn interfaces (input contracts)
│   │   └── out                  → PortOut interfaces (output contracts)
│   └── service                  → Use cases — implements PortIn, calls PortOut
├── config                       → Spring Boot config classes
└── domain                       → Pure Java POJOs — zero framework annotations
```

---

## 4. Architecture Rules — NON-NEGOTIABLE

### Layer responsibilities
- **Controller** → parse HTTP request, call PortIn, return ResponseEntity. Nothing else.
- **OpenApi interface** → declares the endpoint contract + security + OpenAPI docs.
- **PortIn** → interface the controller calls. Implemented by the use case.
- **UseCase** → all business logic. Calls PortOut only — never repositories directly.
- **PortOut** → interface the use case calls. Implemented by the persistence service.
- **PersistenceService** → calls the repository, maps to domain, throws typed exceptions.
- **Domain** → plain Java. No `@Entity`, no `@Service`, no Spring annotations at all.

### Golden rules
- Never call a repository from a controller or use case.
- Never skip a layer — always go Controller → UseCase → PersistenceService.
- Never put business logic in a controller or persistence service.
- Domain objects are pure Java — zero framework annotations.
- For every new endpoint, generate all 6 layers:
  `Controller → OpenApi → PortIn → UseCase → PortOut → PersistenceService`

---

## 5. Naming Conventions

| Layer               | Pattern                      | Example                        |
|---------------------|------------------------------|--------------------------------|
| Controller          | `{Domain}Controller`         | `ClientController`             |
| OpenAPI Interface   | `{Domain}OpenApi`            | `ClientOpenApi`                |
| Input Port          | `{Domain}PortIn`             | `ClientPortIn`                 |
| Output Port         | `{Domain}PersistencePortOut` | `ClientPersistencePortOut`     |
| Use Case            | `{Domain}UseCase`            | `ClientUseCase`                |
| Persistence Service | `{Domain}PersistenceService` | `ClientPersistenceService`     |
| Repository          | `{Entity}Repository`         | `ClientRepository`             |
| Domain Model        | `{Domain}Domain`             | `ClientDomain`                 |
| Response Model      | `{Domain}ResponseModel`      | `ClientSummaryResponseModel`   |
| Query Result DTO    | `{Feature}QueryResultDTO`    | `ClientSummaryQueryResultDTO`  |
| Controller Mapper   | `{Domain}ControllerMapper`   | `ClientControllerMapper`       |
| Persistence Mapper  | `{Domain}PersistenceMapper`  | `ClientPersistenceMapper`      |

---

## 6. Code Conventions

### Mappers
- Always use **MapStruct**. Never map fields manually.
- All mappers extend `BaseMapper`.
- Use `INSTANCE = Mappers.getMapper(...)` singleton pattern.
- Use `@Mapping(qualifiedByName = "...")` for custom conversions.

```java
@Mapper
public interface ClientControllerMapper extends BaseMapper {
    ClientControllerMapper INSTANCE = Mappers.getMapper(ClientControllerMapper.class);
    ClientSummaryResponseModel toResponse(ClientDomain domain);
}
```

### Response Models
- Use Java **records** — immutable, concise, built-in equals/hashCode.
- Annotate every field with `@Schema(description = "...", example = "...")`.
- Use `@JsonFormat(pattern = "yyyy-MM-dd")` for `LocalDate` fields.
- Nested objects are inner records inside the parent record.

### Entities
- All entities extend `BaseEntity` (provides `@Id Long id`).
- Use `@AttributeOverride` to map the correct column name for the ID.
- Always implement `equals()` and `hashCode()` based on `getId()`.
- Table naming: `@Table(name = "T_TABLENAME")`, `@Column(name = "COLUMNNAME")`.

### Repositories
- Use `nativeQuery = true` for complex joins or Oracle-style column naming.
- Query result projections are interfaces in the `persistence/dto` package.

### Logging (Lombok `@Slf4j`)

| Situation                        | Level         |
|----------------------------------|---------------|
| Request received (controller)    | `log.info()`  |
| Success (usecase, persistence)   | `log.info()`  |
| Entity not found                 | `log.warn()`  |
| Unexpected exception             | `log.error()` |

Always include the entity ID or relevant identifier in the message:
```java
log.info("Request received to find client summary: {}", idClient);
log.warn("Client not found in database: {}", idClient);
```

### Error Handling
- Use `NotFoundException` when an entity is not found in persistence.
- Always rethrow typed exceptions — never swallow errors silently.
- Catch `NotFoundException` in persistence services and rethrow with a clear message.

---

## 7. Security

- Every endpoint in the OpenApi interface **must** have `@PreAuthorize`.
- Roles are constants in the `Authorities` abstract class.
- Follow the role hierarchy: specific role → group role → ADMIN.

```java
// In Authorities.java
public static final String ROLE_CLIENT_QUERY_SUMMARY =
    "hasAnyAuthority('ROLE_CLIENT_QUERY_SUMMARY', " + ROLE_CLIENT_QUERY + ")";

// On the OpenApi method
@PreAuthorize(Authorities.ROLE_CLIENT_QUERY_SUMMARY)
```

---

## 8. OpenAPI Documentation

Every OpenApi interface must have:
- `@Tag(name = "...", description = "...")` on the interface
- `@Operation(summary = "...", description = "...")` on every method
- `@ApiResponse` for every expected status code (200, 404, etc.)
- `@DefaultErrorApiResponses` for standard error codes
- `@Parameter(description, required, example)` on every path/query param

---

## 9. Reference — Full 6-Layer Flow (GET endpoint)

Use this as the canonical template for every new GET endpoint.

### Controller
```java
@Slf4j
@RestController
@RequiredArgsConstructor
public class ClientController implements ClientOpenApi {

    private final ClientPortIn clientPortIn;

    @Override
    public ResponseEntity<ClientSummaryResponseModel> findSummary(@PathVariable Long idClient) {
        log.info("Request received to find client summary: {}", idClient);
        ClientSummaryResponseModel response = ClientControllerMapper.INSTANCE
            .toResponse(clientPortIn.findSummaryById(idClient));
        log.info("Client {} summary returned successfully", idClient);
        return ResponseEntity.ok(response);
    }
}
```

### OpenApi Interface
```java
@RequestMapping(value = "/clients", produces = MediaType.APPLICATION_JSON_VALUE)
@Tag(name = "Client", description = "Endpoints for managing client data")
public interface ClientOpenApi extends OpenApi {

    @PreAuthorize(Authorities.ROLE_CLIENT_QUERY_SUMMARY)
    @GetMapping("/{idClient}/summary")
    @Operation(summary = "Find client summary", description = "Returns client summary by ID")
    @ApiResponse(responseCode = "200", description = "Data returned successfully",
        content = @Content(schema = @Schema(implementation = ClientSummaryResponseModel.class)))
    @ApiResponse(responseCode = "404", description = "Client not found",
        content = @Content(mediaType = "application/json"))
    @DefaultErrorApiResponses
    ResponseEntity<ClientSummaryResponseModel> findSummary(
        @Parameter(description = "Client ID", required = true, example = "1001")
        Long idClient
    );
}
```

### Port In
```java
public interface ClientPortIn extends PortIn {
    ClientDomain findSummaryById(Long idClient);
}
```

### Use Case
```java
@Slf4j
@Service
@RequiredArgsConstructor
public class ClientUseCase implements ClientPortIn {

    private final ClientPersistencePortOut clientPersistencePortOut;

    @Override
    public ClientDomain findSummaryById(Long idClient) {
        log.info("Finding client summary, id: {}", idClient);
        return clientPersistencePortOut.findSummaryById(idClient);
    }
}
```

### Port Out
```java
public interface ClientPersistencePortOut extends PortOut {
    ClientDomain findSummaryById(Long idClient);
}
```

### Persistence Service
```java
@Slf4j
@Service
@RequiredArgsConstructor
public class ClientPersistenceService implements ClientPersistencePortOut {

    private final ClientRepository clientRepository;

    @Override
    public ClientDomain findSummaryById(Long idClient) {
        log.info("Finding client summary in database: {}", idClient);
        Optional<ClientSummaryQueryResultDTO> result = clientRepository.findSummaryById(idClient);

        if (result.isEmpty()) {
            log.warn("Client not found in database: {}", idClient);
            throw new NotFoundException("Client not found for the given code.");
        }

        ClientDomain domain = ClientPersistenceMapper.INSTANCE.toDomain(result.get());
        log.info("Client {} summary retrieved successfully", idClient);
        return domain;
    }
}
```

---

## 10. Boundaries — Safety Rules

### Always do
- Follow all 6 hexagonal layers for every new endpoint.
- Add `@PreAuthorize` to every OpenApi method.
- Use MapStruct for all mappings — never map fields manually.
- Add `log.info`, `log.warn`, `log.error` at the correct layers.
- Run `./mvnw test` before considering a task done.

### Ask before doing
- Adding a new Maven dependency to `pom.xml`.
- Creating a file not listed in the current task's "Files to Touch".
- Modifying `BaseEntity`, `BaseMapper`, or any shared base class.
- Changing the database schema or existing native queries.
- Refactoring more than one domain at the same time.

### Never do
- Call a repository from a controller or use case.
- Put business logic in a controller or persistence service.
- Add Spring/JPA annotations to domain classes.
- Map fields manually — always use MapStruct.
- Swallow exceptions silently.
- Create an endpoint without `@PreAuthorize`.
- Skip the OpenAPI documentation annotations.
