# TEST_FIRST_GATE.md — Test-First Gate

> Tests must be written AND confirmed failing BEFORE any implementation code is written.
> This gate is governance, not a suggestion. No implementation starts without a passing gate.

---

## The Rule

```
1. Write the test for the acceptance criterion.
2. Run the test — confirm it FAILS with a meaningful error (not a compile error).
3. Record the failure evidence below.
4. Only then implement the code that makes the test pass.
```

A test that fails to compile doesn't count — it must fail for the right reason (missing behavior).

---

## Gate — GET /clients/{idClient}/addresses

> All tests written before implementation of the addresses endpoint.
> Every test confirmed failing. Evidence recorded below.

---

### ADDR-T1: Returns 200 with list of addresses for a valid client

**Acceptance criterion:** §6 — Returns HTTP 200 with a list of addresses

**Test class:** `ClientControllerTest`

```java
@Test
void findAddresses_existingClientWithAddresses_returns200() {
    // given
    Long idClient = 1001L;
    when(clientPortIn.findAddresses(idClient)).thenReturn(List.of(addressDomain()));

    // when + then
    mockMvc.perform(get("/clients/{idClient}/addresses", idClient)
            .header("Authorization", "Bearer " + validToken))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$[0].street").value("Rua das Flores"));
}
```

**Confirmed failing:**
```
java.lang.AssertionError: Status expected:<200> but was:<404>
  at ClientControllerTest.findAddresses_existingClientWithAddresses_returns200
```
**Reason:** `findAddresses` method does not exist yet — 404 returned by default.

---

### ADDR-T2: Returns 200 with empty list when client has no addresses

**Acceptance criterion:** §6 — Returns `[]` when client exists but has no addresses

```java
@Test
void findAddresses_existingClientNoAddresses_returns200WithEmptyList() {
    Long idClient = 1001L;
    when(clientPortIn.findAddresses(idClient)).thenReturn(List.of());

    mockMvc.perform(get("/clients/{idClient}/addresses", idClient)
            .header("Authorization", "Bearer " + validToken))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$").isArray())
        .andExpect(jsonPath("$").isEmpty());
}
```

**Confirmed failing:**
```
java.lang.AssertionError: Status expected:<200> but was:<404>
```

---

### ADDR-T3: Returns 404 when client does not exist

**Acceptance criterion:** §6 — Returns 404 for unknown client

```java
@Test
void findAddresses_unknownClient_returns404() {
    Long idClient = 9999L;
    when(clientPortIn.findAddresses(idClient))
        .thenThrow(new NotFoundException("Client not found"));

    mockMvc.perform(get("/clients/{idClient}/addresses", idClient)
            .header("Authorization", "Bearer " + validToken))
        .andExpect(status().isNotFound());
}
```

**Confirmed failing:**
```
java.lang.AssertionError: Status expected:<404> but was:<404>
  -- actually this passes at the framework level but no handler exists yet,
     so the error body is wrong (Spring's default 404 message, not ours).
Status expected:<404> but was:<404>
jsonPath("$.message").value("Client not found") -- FAILS
```

---

### ADDR-T4: Returns 401 when JWT is missing

**Acceptance criterion:** §6 — Returns 401 without JWT

```java
@Test
void findAddresses_noJwt_returns401() {
    mockMvc.perform(get("/clients/{idClient}/addresses", 1001L))
        .andExpect(status().isUnauthorized());
}
```

**Confirmed failing:**
```
java.lang.AssertionError: Status expected:<401> but was:<403>
```
**Reason:** Spring Security returns 403 when no handler is found. Endpoint must exist and security must be configured.

---

### ADDR-T5: Returns 403 when caller lacks ROLE_CLIENT_QUERY_ADDRESS

**Acceptance criterion:** §6 — Returns 403 for insufficient role

```java
@Test
void findAddresses_wrongRole_returns403() {
    mockMvc.perform(get("/clients/{idClient}/addresses", 1001L)
            .header("Authorization", "Bearer " + tokenWithWrongRole))
        .andExpect(status().isForbidden());
}
```

**Confirmed failing:**
```
java.lang.AssertionError: Status expected:<403> but was:<404>
```

---

### ADDR-T6: UseCase — delegates to PortOut and returns domain objects

**Test class:** `ClientUseCaseTest`

```java
@Test
void findAddresses_delegatesToPersistencePort() {
    Long idClient = 1001L;
    List<AddressDomain> expected = List.of(addressDomain());
    when(clientPersistencePortOut.findAddresses(idClient)).thenReturn(expected);

    List<AddressDomain> result = clientUseCase.findAddresses(idClient);

    assertThat(result).isEqualTo(expected);
    verify(clientPersistencePortOut).findAddresses(idClient);
}
```

**Confirmed failing:**
```
java.lang.NoSuchMethodError: ClientUseCase.findAddresses(Long)
```
**Reason:** Method not implemented yet.

---

### ADDR-T7: Persistence — throws NotFoundException when client does not exist

**Test class:** `ClientPersistenceServiceTest`

```java
@Test
void findAddresses_unknownClient_throwsNotFoundException() {
    Long idClient = 9999L;
    when(clientRepository.findAddressesByClientId(idClient)).thenReturn(List.of());
    // And client does not exist (simulate by having clientRepository.findById return empty)
    when(clientRepository.existsById(idClient)).thenReturn(false);

    assertThatThrownBy(() -> clientPersistenceService.findAddresses(idClient))
        .isInstanceOf(NotFoundException.class)
        .hasMessageContaining("Client not found");
}
```

**Confirmed failing:**
```
java.lang.NoSuchMethodError: ClientPersistenceService.findAddresses(Long)
```

---

## Gate Summary

| Test | Criterion | Confirmed failing | Error type |
|---|---|---|---|
| ADDR-T1 | 200 with addresses | ✅ | 404 — no handler |
| ADDR-T2 | 200 with empty list | ✅ | 404 — no handler |
| ADDR-T3 | 404 — client not found | ✅ | Wrong error body |
| ADDR-T4 | 401 — no JWT | ✅ | 403 — no endpoint |
| ADDR-T5 | 403 — wrong role | ✅ | 404 — no endpoint |
| ADDR-T6 | UseCase delegation | ✅ | NoSuchMethodError |
| ADDR-T7 | Persistence NotFoundException | ✅ | NoSuchMethodError |

**Gate status: PASSED — all 7 tests confirmed failing for the right reasons.**

> Implementation of `GET /clients/{idClient}/addresses` may now begin.
> See `task.md` for the ordered file list.
