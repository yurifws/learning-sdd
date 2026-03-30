# task.md — Current Task

> This is the only file you update constantly.
> Change "Active Task" and "Files to Touch" every time you start something new.
> Keep the other files (constitution, spec, plan) stable.

---

## Active Task
Implement `GET /clientes/{idCliente}/enderecos` — Buscar endereços do cliente

---

## Acceptance Criteria
- Authenticated users with `ROLE_CLIENTE_CONSULTA_ENDERECO` can call this endpoint
- Returns a list of addresses (`List<EnderecoResponseModel>`) for the given customer ID
- Returns `404` with a clear message if the customer does not exist
- Returns `200` with an empty list if the customer exists but has no registered addresses

---

## Context
- The customer resumo flow is already implemented — follow the exact same pattern (see `spec.md`)
- `ClienteOpenApi` interface already exists — add the new method there
- `ClienteController` already exists — add the new method there
- New role constant must be added to `Authorities`

---

## Files to Create / Touch

| File | Action |
|---|---|
| `Authorities.java` | Add `ROLE_CLIENTE_CONSULTA_ENDERECO` constant |
| `ClientePortIn.java` | Add `buscarEnderecos(Long idCliente)` method |
| `ClienteUseCase.java` | Implement `buscarEnderecos` |
| `ClientePersistencePortOut.java` | Add `buscarEnderecos(Long idCliente)` method |
| `ClientePersistenceService.java` | Implement `buscarEnderecos` with repository call |
| `EnderecoQueryResultDTO.java` | New: query result projection interface |
| `EnderecoDomain.java` | New: domain model |
| `EnderecoResponseModel.java` | New: response record with `@Schema` |
| `ClientePersistenceMapper.java` | Add `toDomain(EnderecoQueryResultDTO)` |
| `ClienteControllerMapper.java` | Add `toResponse(EnderecoDomain)` |
| `ClienteRepository.java` | Add `buscarEnderecosPorIdCliente(@Param Long idCliente)` |
| `ClienteOpenApi.java` | Add `buscarEnderecos` method signature |
| `ClienteController.java` | Add `buscarEnderecos` implementation |

---

## Done When
- [ ] Endpoint returns `200` with list of addresses for a valid customer
- [ ] Endpoint returns `200` with empty list when customer has no addresses
- [ ] Endpoint returns `404` when customer ID does not exist
- [ ] `@PreAuthorize` is set correctly on the OpenApi method
- [ ] All layers follow the hexagonal pattern from `spec.md`
- [ ] All mappers use MapStruct (no manual mapping)
- [ ] Logs present at controller, usecase, and persistence levels

---

## Notes / Blockers
- [ ] Define the fields for `EnderecoResponseModel` before starting (logradouro, numero, bairro, cidade, estado, cep?)
- [ ] Confirm if address table has a FK to the customer table or uses a join table
