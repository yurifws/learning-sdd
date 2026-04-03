# task.md — Current Task

> This is the only file you update constantly.
> Change "Active Task" and "Files to Touch" every time you start something new.
> Keep the other files (constitution, spec, plan) stable.

---

## Active Task
Implement `GET /clients/{idClient}/addresses` — Find client addresses

---

## Acceptance Criteria
- Authenticated users with `ROLE_CLIENT_QUERY_ADDRESS` can call this endpoint
- Returns a list of addresses (`List<AddressResponseModel>`) for the given client ID
- Returns `404` with a clear message if the client does not exist
- Returns `200` with an empty list if the client exists but has no registered addresses

---

## Context
- The client summary flow is already implemented — follow the exact same pattern (see `spec.md`)
- `ClientOpenApi` interface already exists — add the new method there
- `ClientController` already exists — add the new method there
- New role constant must be added to `Authorities`

---

## Files to Create / Touch

| File | Action |
|---|---|
| `Authorities.java` | Add `ROLE_CLIENT_QUERY_ADDRESS` constant |
| `ClientPortIn.java` | Add `findAddresses(Long idClient)` method |
| `ClientUseCase.java` | Implement `findAddresses` |
| `ClientPersistencePortOut.java` | Add `findAddresses(Long idClient)` method |
| `ClientPersistenceService.java` | Implement `findAddresses` with repository call |
| `AddressQueryResultDTO.java` | New: query result projection interface |
| `AddressDomain.java` | New: domain model |
| `AddressResponseModel.java` | New: response record with `@Schema` |
| `ClientPersistenceMapper.java` | Add `toDomain(AddressQueryResultDTO)` |
| `ClientControllerMapper.java` | Add `toResponse(AddressDomain)` |
| `ClientRepository.java` | Add `findAddressesByClientId(@Param Long idClient)` |
| `ClientOpenApi.java` | Add `findAddresses` method signature |
| `ClientController.java` | Add `findAddresses` implementation |

---

## Done When
- [ ] Endpoint returns `200` with list of addresses for a valid client
- [ ] Endpoint returns `200` with empty list when client has no addresses
- [ ] Endpoint returns `404` when client ID does not exist
- [ ] `@PreAuthorize` is set correctly on the OpenApi method
- [ ] All layers follow the hexagonal pattern from `spec.md`
- [ ] All mappers use MapStruct (no manual mapping)
- [ ] Logs present at controller, usecase, and persistence levels

---

## Notes / Blockers
- [ ] Define the fields for `AddressResponseModel` before starting (street, number, neighborhood, city, state, zipCode?)
- [ ] Confirm if address table has a FK to the client table or uses a join table
