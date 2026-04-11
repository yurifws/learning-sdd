# spec.md — Project Specification

## Project
**tech.example.api.client**
A learning project demonstrating Hexagonal Architecture (Ports & Adapters) with Spring Boot.
This is a REST API for managing client data — used as a reference implementation for SDD (Software-Driven Development) with AI.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Language | Java 17+ |
| Framework | Spring Boot |
| Persistence | Spring Data JPA + H2 (dev) / PostgreSQL (prod) |
| Security | Spring Security + `@PreAuthorize` |
| Mapping | MapStruct |
| Documentation | OpenAPI 3 (Swagger) |
| gRPC | Optional input adapter |

> Note: Oracle-style table naming (`T_TABLENAME`, `IDCOLUMN`) is kept intentionally
> to mirror enterprise patterns common in real-world projects.

---

## Package Structure

```
tech.example.api.client
├── adapters
│   ├── input
│   │   ├── controller
│   │   │   ├── constant
│   │   │   ├── dto
│   │   │   ├── enums
│   │   │   ├── mapper
│   │   │   ├── openapi
│   │   │   │   └── apiresponses
│   │   │   └── security
│   │   └── grpc
│   │       └── mapper
│   └── output
│       └── persistence
│           ├── dto
│           ├── entity
│           ├── exceptions
│           ├── mapper
│           ├── repository
│           └── service
├── application
│   ├── ports
│   │   ├── in
│   │   └── out
│   └── service
├── config
└── domain
```

---

## Implemented Endpoints

### GET /clients/{idClient}/summary
- **Auth:** `ROLE_CLIENT_QUERY_SUMMARY`
- **Returns:** Client summary data
- **Response Model:** `ClientSummaryResponseModel`
- **Status:** ✅ Done
- **Errors:** `404` client not found

### GET /clients/{idClient}/addresses
- **Auth:** `ROLE_CLIENT_QUERY_ADDRESS`
- **Returns:** List of client addresses — empty list `[]` if none registered
- **Response Model:** `List<AddressResponseModel>`
- **Status:** ⏳ Pending
- **Errors:** `404` client not found

---

## Reference Implementation

### Full flow for a GET endpoint — all 6 layers:

**1. Controller**
```java
@Slf4j
@RestController
@RequiredArgsConstructor
public class ClientController implements ClientOpenApi {

    private final ClientPortIn clientPortIn;

    @Override
    public ResponseEntity<ClientSummaryResponseModel> findSummary(@PathVariable Long idClient) {
        log.info("Request received to find client summary: {}", idClient);
        ClientSummaryResponseModel response = ClientControllerMapper.INSTANCE.toResponse(
            clientPortIn.findSummaryById(idClient)
        );
        log.info("Client {} summary returned successfully", idClient);
        return ResponseEntity.ok(response);
    }
}
```

**2. OpenApi Interface**
```java
@RequestMapping(value = "/clients", produces = MediaType.APPLICATION_JSON_VALUE)
@Tag(name = "Client", description = "Endpoints for managing client data")
public interface ClientOpenApi extends OpenApi {

    @PreAuthorize(Authorities.ROLE_CLIENT_QUERY_SUMMARY)
    @GetMapping("/{idClient}/summary")
    @Operation(summary = "Find client summary", description = "Returns client summary by ID")
    @ApiResponse(responseCode = "200", description = "Data returned successfully",
        content = @Content(mediaType = APPLICATION_JSON_VALUE,
            schema = @Schema(implementation = ClientSummaryResponseModel.class)))
    @ApiResponse(responseCode = "404", description = "Client not found",
        content = @Content(mediaType = "application/json"))
    @DefaultErrorApiResponses
    ResponseEntity<ClientSummaryResponseModel> findSummary(
        @Parameter(description = "Client ID", required = true, example = "1001")
        Long idClient
    );
}
```

**3. Port In**
```java
public interface ClientPortIn extends PortIn {
    ClientDomain findSummaryById(Long idClient);
}
```

**4. Use Case**
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

**5. Port Out**
```java
public interface ClientPersistencePortOut extends PortOut {
    ClientDomain findSummaryById(Long idClient);
}
```

**6. Persistence Service**
```java
@Slf4j
@Service
@RequiredArgsConstructor
public class ClientPersistenceService implements ClientPersistencePortOut {
    private final ClientRepository clientRepository;

    @Override
    public ClientDomain findSummaryById(Long idClient) {
        log.info("Finding client summary in database: {}", idClient);

        Optional<ClientSummaryQueryResultDTO> resultOpt =
            clientRepository.findSummaryById(idClient);

        if (resultOpt.isEmpty()) {
            log.warn("Client not found in database: {}", idClient);
            throw new NotFoundException("Client not found for the given code.");
        }

        ClientDomain domain = ClientPersistenceMapper.INSTANCE.toDomain(resultOpt.get());
        log.info("Client {} summary retrieved successfully", idClient);
        return domain;
    }
}
```

---

## Domain Model

```java
@Getter @Setter @NoArgsConstructor
public class ClientDomain {
    private Long id;
    private String name;
    private String email;
    private String cpf;
    private LocalDate birthDate;
    private String status;
    private PlanDomain plan;

    @Getter @Setter @NoArgsConstructor
    public static class PlanDomain {
        private Long id;
        private String description;
    }
}
```

---

## Response Model

```java
@Schema(description = "Client summary")
public record ClientSummaryResponseModel(

    @Schema(description = "Client ID", example = "1001")
    Long id,

    @Schema(description = "Client name", example = "John Smith")
    String name,

    @Schema(description = "Client email", example = "john@email.com")
    String email,

    @Schema(description = "Client CPF", example = "12345678900")
    String cpf,

    @Schema(description = "Birth date", example = "1990-01-15")
    @JsonFormat(pattern = "yyyy-MM-dd")
    LocalDate birthDate,

    @Schema(description = "Client status", example = "A")
    String status,

    @Schema(description = "Client plan")
    PlanResponseModel plan
) {
    @Schema(description = "Client plan data")
    public record PlanResponseModel(
        @Schema(description = "Plan ID", example = "3")
        Long id,
        @Schema(description = "Plan description", example = "Premium Plan")
        String description
    ) {}
}
```
