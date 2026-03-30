# spec.md — Project Specification

## Project
**tech.example.api.cliente**
A learning project demonstrating Hexagonal Architecture (Ports & Adapters) with Spring Boot.
This is a REST API for managing customer (cliente) data — used as a reference implementation for SDD (Software-Driven Development) with AI.

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
tech.example.api.cliente
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

### GET /clientes/{idCliente}/resumo
- **Auth:** `ROLE_CLIENTE_CONSULTA_RESUMO`
- **Returns:** Customer summary data
- **Response Model:** `ClienteResumoResponseModel`
- **Status:** ✅ Done

### GET /clientes/{idCliente}/enderecos
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
public class ClienteController implements ClienteOpenApi {

    private final ClientePortIn clientePortIn;

    @Override
    public ResponseEntity<ClienteResumoResponseModel> buscarResumo(@PathVariable Long idCliente) {
        log.info("Requisição recebida para buscar resumo do cliente: {}", idCliente);
        ClienteResumoResponseModel response = ClienteControllerMapper.INSTANCE.toResponse(
            clientePortIn.buscarResumoPorId(idCliente)
        );
        log.info("Resumo do cliente {} retornado com sucesso", idCliente);
        return ResponseEntity.ok(response);
    }
}
```

**2. OpenApi Interface**
```java
@RequestMapping(value = "/clientes", produces = MediaType.APPLICATION_JSON_VALUE)
@Tag(name = "Cliente", description = "Endpoints para gerenciamento de dados de clientes")
public interface ClienteOpenApi extends OpenApi {

    @PreAuthorize(Authorities.ROLE_CLIENTE_CONSULTA_RESUMO)
    @GetMapping("/{idCliente}/resumo")
    @Operation(summary = "Buscar resumo do cliente", description = "Retorna resumo do cliente pelo ID")
    @ApiResponse(responseCode = "200", description = "Dados retornados com sucesso",
        content = @Content(mediaType = APPLICATION_JSON_VALUE,
            schema = @Schema(implementation = ClienteResumoResponseModel.class)))
    @ApiResponse(responseCode = "404", description = "Cliente não encontrado",
        content = @Content(mediaType = "application/json"))
    @DefaultErrorApiResponses
    ResponseEntity<ClienteResumoResponseModel> buscarResumo(
        @Parameter(description = "ID do cliente", required = true, example = "1001")
        Long idCliente
    );
}
```

**3. Port In**
```java
public interface ClientePortIn extends PortIn {
    ClienteDomain buscarResumoPorId(Long idCliente);
}
```

**4. Use Case**
```java
@Slf4j
@Service
@RequiredArgsConstructor
public class ClienteUseCase implements ClientePortIn {
    private final ClientePersistencePortOut clientePersistencePortOut;

    @Override
    public ClienteDomain buscarResumoPorId(Long idCliente) {
        log.info("Buscando resumo do cliente id: {}", idCliente);
        return clientePersistencePortOut.buscarResumoPorId(idCliente);
    }
}
```

**5. Port Out**
```java
public interface ClientePersistencePortOut extends PortOut {
    ClienteDomain buscarResumoPorId(Long idCliente);
}
```

**6. Persistence Service**
```java
@Slf4j
@Service
@RequiredArgsConstructor
public class ClientePersistenceService implements ClientePersistencePortOut {
    private final ClienteRepository clienteRepository;

    @Override
    public ClienteDomain buscarResumoPorId(Long idCliente) {
        log.info("Buscando resumo do cliente no banco de dados: {}", idCliente);

        Optional<ClienteResumoQueryResultDTO> resultOpt =
            clienteRepository.buscarResumoPorId(idCliente);

        if (resultOpt.isEmpty()) {
            log.warn("Cliente não encontrado no banco: {}", idCliente);
            throw new NotFoundException("Cliente não encontrado para o código informado.");
        }

        ClienteDomain domain = ClientePersistenceMapper.INSTANCE.toDomain(resultOpt.get());
        log.info("Resumo do cliente {} recuperado com sucesso", idCliente);
        return domain;
    }
}
```

---

## Domain Model

```java
@Getter @Setter @NoArgsConstructor
public class ClienteDomain {
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
@Schema(description = "Resumo do cliente")
public record ClienteResumoResponseModel(

    @Schema(description = "ID do cliente", example = "1001")
    Long id,

    @Schema(description = "Nome do cliente", example = "João da Silva")
    String nome,

    @Schema(description = "Email do cliente", example = "joao@email.com")
    String email,

    @Schema(description = "CPF do cliente", example = "12345678900")
    String cpf,

    @Schema(description = "Data de nascimento", example = "1990-01-15")
    @JsonFormat(pattern = "yyyy-MM-dd")
    LocalDate dataNascimento,

    @Schema(description = "Status do cliente", example = "A")
    String status,

    @Schema(description = "Plano do cliente")
    PlanoResponseModel plano
) {
    @Schema(description = "Dados do plano do cliente")
    public record PlanoResponseModel(
        @Schema(description = "ID do plano", example = "3")
        Long id,
        @Schema(description = "Descrição do plano", example = "Plano Premium")
        String descricao
    ) {}
}
```
