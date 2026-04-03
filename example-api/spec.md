# spec.md — Project Specification

## Project
**tech.example.api.client**
A learning project demonstrating Hexagonal Architecture (Ports & Adapters) with Spring Boot.
This is a REST API for managing customer (client) data — used as a reference implementation for SDD (Software-Driven Development) with AI.

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

### GET /clients/{idClient}/resumo
- **Auth:** `ROLE_CLIENTE_CONSULTA_RESUMO`
- **Returns:** Customer summary data
- **Response Model:** `ClientResumoResponseModel`
- **Status:** ✅ Done

### GET /clients/{idClient}/enderecos
- **Auth:** `ROLE_CLIENTE_CONSULTA_ENDERECO`
- **Returns:** List of customer addresses
- **Response Model:** `List<EnderecoResponseModel>`
- **Status:** ⏳ Pending

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
    public ResponseEntity<ClientResumoResponseModel> buscarResumo(@PathVariable Long idClient) {
        log.info("Requisição recebida para buscar resumo do client: {}", idClient);
        ClientResumoResponseModel response = ClientControllerMapper.INSTANCE.toResponse(
            clientPortIn.buscarResumoPorId(idClient)
        );
        log.info("Resumo do client {} retornado com sucesso", idClient);
        return ResponseEntity.ok(response);
    }
}
```

**2. OpenApi Interface**
```java
@RequestMapping(value = "/clients", produces = MediaType.APPLICATION_JSON_VALUE)
@Tag(name = "Client", description = "Endpoints para gerenciamento de dados de clients")
public interface ClientOpenApi extends OpenApi {

    @PreAuthorize(Authorities.ROLE_CLIENTE_CONSULTA_RESUMO)
    @GetMapping("/{idClient}/resumo")
    @Operation(summary = "Buscar resumo do client", description = "Retorna resumo do client pelo ID")
    @ApiResponse(responseCode = "200", description = "Dados retornados com sucesso",
        content = @Content(mediaType = APPLICATION_JSON_VALUE,
            schema = @Schema(implementation = ClientResumoResponseModel.class)))
    @ApiResponse(responseCode = "404", description = "Client não encontrado",
        content = @Content(mediaType = "application/json"))
    @DefaultErrorApiResponses
    ResponseEntity<ClientResumoResponseModel> buscarResumo(
        @Parameter(description = "ID do client", required = true, example = "1001")
        Long idClient
    );
}
```

**3. Port In**
```java
public interface ClientPortIn extends PortIn {
    ClientDomain buscarResumoPorId(Long idClient);
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
    public ClientDomain buscarResumoPorId(Long idClient) {
        log.info("Buscando resumo do client id: {}", idClient);
        return clientPersistencePortOut.buscarResumoPorId(idClient);
    }
}
```

**5. Port Out**
```java
public interface ClientPersistencePortOut extends PortOut {
    ClientDomain buscarResumoPorId(Long idClient);
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
    public ClientDomain buscarResumoPorId(Long idClient) {
        log.info("Buscando resumo do client no banco de dados: {}", idClient);

        Optional<ClientResumoQueryResultDTO> resultOpt =
            clientRepository.buscarResumoPorId(idClient);

        if (resultOpt.isEmpty()) {
            log.warn("Client não encontrado no banco: {}", idClient);
            throw new NotFoundException("Client não encontrado para o código informado.");
        }

        ClientDomain domain = ClientPersistenceMapper.INSTANCE.toDomain(resultOpt.get());
        log.info("Resumo do client {} recuperado com sucesso", idClient);
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
    private String nome;
    private String email;
    private String cpf;
    private LocalDate dataNascimento;
    private String status;
    private PlanoDomain plano;

    @Getter @Setter @NoArgsConstructor
    public static class PlanoDomain {
        private Long id;
        private String descricao;
    }
}
```

---

## Response Model

```java
@Schema(description = "Resumo do client")
public record ClientResumoResponseModel(

    @Schema(description = "ID do client", example = "1001")
    Long id,

    @Schema(description = "Nome do client", example = "João da Silva")
    String nome,

    @Schema(description = "Email do client", example = "joao@email.com")
    String email,

    @Schema(description = "CPF do client", example = "12345678900")
    String cpf,

    @Schema(description = "Data de nascimento", example = "1990-01-15")
    @JsonFormat(pattern = "yyyy-MM-dd")
    LocalDate dataNascimento,

    @Schema(description = "Status do client", example = "A")
    String status,

    @Schema(description = "Plano do client")
    PlanoResponseModel plano
) {
    @Schema(description = "Dados do plano do client")
    public record PlanoResponseModel(
        @Schema(description = "ID do plano", example = "3")
        Long id,
        @Schema(description = "Descrição do plano", example = "Plano Premium")
        String descricao
    ) {}
}
```
