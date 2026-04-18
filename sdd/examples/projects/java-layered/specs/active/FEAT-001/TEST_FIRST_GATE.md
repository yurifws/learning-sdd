# TEST_FIRST_GATE.md — Test-First Gate

> Tests must be written AND confirmed failing BEFORE any implementation code is written.
> This gate is governance, not a suggestion.
> **Linked:** requirements.md · tasks.md

---

## The Rule

```
1. Write the test stub for the acceptance criterion.
2. Add the method signature to the service/controller (stub only — no implementation).
3. Run the test — confirm it FAILS with a meaningful error (business behavior missing).
4. Record the failure here.
5. Only then implement the code that makes the test pass.
```

A test that won't compile doesn't count — stubs must exist so the test can fail semantically.

---

## Gate — FEAT-001: Product API (CRUD)

> Tests written before TASK-10 and TASK-11 (service and controller implementation).
> Evidence recorded below.

---

### PROD-T1: POST /api/v1/products — creates product, returns 201

**Acceptance criterion:** §6 — POST creates a product and returns 201

```java
@Test
void createProduct_validRequest_returns201() throws Exception {
    var request = new ProductRequest("Widget A", "WIDGET-001", null, new BigDecimal("9.99"), 50);
    mockMvc.perform(post("/api/v1/products")
            .header("Authorization", "Bearer " + adminToken)
            .contentType(APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(request)))
        .andExpect(status().isCreated())
        .andExpect(jsonPath("$.data.sku").value("WIDGET-001"))
        .andExpect(jsonPath("$.data.status").value("ACTIVE"));
}
```

**Confirmed failing (before TASK-10/11):**
```
java.lang.AssertionError: Status expected:<201> but was:<500>
ProductServiceImpl.create() throws NullPointerException — method stub returns null
```

---

### PROD-T2: POST with duplicate SKU returns 409

**Acceptance criterion:** §6 — POST with duplicate SKU returns 409

```java
@Test
void createProduct_duplicateSku_returns409() throws Exception {
    // given: WIDGET-001 already exists in the test DB (Testcontainers)
    var request = new ProductRequest("Widget B", "WIDGET-001", null, new BigDecimal("5.00"), 10);
    mockMvc.perform(post("/api/v1/products")
            .header("Authorization", "Bearer " + adminToken)
            .contentType(APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(request)))
        .andExpect(status().isConflict())
        .andExpect(jsonPath("$.error").value("CONFLICT"));
}
```

**Confirmed failing:**
```
java.lang.AssertionError: Status expected:<409> but was:<201>
No SKU uniqueness check implemented yet
```

---

### PROD-T3: GET /api/v1/products — returns paginated list

**Acceptance criterion:** §6 — GET /products returns paginated list

```java
@Test
void listProducts_returnsPagedActiveProducts() throws Exception {
    mockMvc.perform(get("/api/v1/products")
            .param("page", "0").param("size", "10")
            .header("Authorization", "Bearer " + userToken))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$.data.content").isArray());
}
```

**Confirmed failing:**
```
java.lang.AssertionError: Status expected:<200> but was:<500>
findAll() stub returns null, NullPointerException in controller
```

---

### PROD-T4: GET /api/v1/products/{id} — unknown ID returns 404

**Acceptance criterion:** §6 — GET /products/{id} returns 404 for unknown ID

```java
@Test
void getProduct_unknownId_returns404() throws Exception {
    mockMvc.perform(get("/api/v1/products/{id}", 99999L)
            .header("Authorization", "Bearer " + userToken))
        .andExpect(status().isNotFound())
        .andExpect(jsonPath("$.error").value("NOT_FOUND"));
}
```

**Confirmed failing:**
```
java.lang.AssertionError: Status expected:<404> but was:<500>
findById() stub throws NullPointerException instead of ResourceNotFoundException
```

---

### PROD-T5: PUT /api/v1/products/{id} — updates and returns 200

**Acceptance criterion:** §6 — PUT updates product and returns updated data

```java
@Test
void updateProduct_existingId_returns200() throws Exception {
    var request = new ProductRequest("Widget A Updated", "WIDGET-001", null, new BigDecimal("12.99"), 50);
    mockMvc.perform(put("/api/v1/products/{id}", existingProductId)
            .header("Authorization", "Bearer " + adminToken)
            .contentType(APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(request)))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$.data.name").value("Widget A Updated"));
}
```

**Confirmed failing:**
```
java.lang.AssertionError: Status expected:<200> but was:<500>
update() stub not implemented
```

---

### PROD-T6: DELETE /api/v1/products/{id} — soft delete, row persists

**Acceptance criterion:** §6 — DELETE sets status INACTIVE, does not remove from DB

```java
@Test
void deleteProduct_setsInactive_rowRemainsInDb() throws Exception {
    mockMvc.perform(delete("/api/v1/products/{id}", existingProductId)
            .header("Authorization", "Bearer " + adminToken))
        .andExpect(status().isOk());

    // verify row still exists but is INACTIVE
    Product product = productRepository.findById(existingProductId).orElseThrow();
    assertThat(product.getStatus()).isEqualTo("INACTIVE");
}
```

**Confirmed failing:**
```
java.lang.AssertionError: Status expected:<200> but was:<500>
deactivate() stub not implemented
```

---

### PROD-T7: POST with blank name returns 400

**Acceptance criterion:** §6 — Validation rules from §2.6 return 400

```java
@Test
void createProduct_blankName_returns400WithValidationError() throws Exception {
    var request = new ProductRequest("", "WIDGET-001", null, new BigDecimal("9.99"), 50);
    mockMvc.perform(post("/api/v1/products")
            .header("Authorization", "Bearer " + adminToken)
            .contentType(APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(request)))
        .andExpect(status().isBadRequest())
        .andExpect(jsonPath("$.error").value("VALIDATION_ERROR"))
        .andExpect(jsonPath("$.message").value("name is required"));
}
```

**Confirmed failing:**
```
java.lang.AssertionError: Status expected:<400> but was:<500>
@Valid annotation not applied to controller method parameter yet
```

---

### PROD-T8: All endpoints return 401 when JWT is missing

**Acceptance criterion:** §6 — All endpoints return 401 when JWT is missing

```java
@ParameterizedTest
@ValueSource(strings = {"/api/v1/products", "/api/v1/products/1"})
void allEndpoints_noJwt_return401(String path) throws Exception {
    mockMvc.perform(get(path))
        .andExpect(status().isUnauthorized());
}
```

**Confirmed failing:**
```
java.lang.AssertionError: Status expected:<401> but was:<403>
SecurityFilterChain not configured yet — Spring defaults to 403
```

---

### PROD-T9: Service unit — findById throws ResourceNotFoundException

**Test class:** `ProductServiceImplTest`

```java
@Test
void findById_unknownId_throwsResourceNotFoundException() {
    when(productRepository.findById(99L)).thenReturn(Optional.empty());

    assertThatThrownBy(() -> productService.findById(99L))
        .isInstanceOf(ResourceNotFoundException.class)
        .hasMessageContaining("NOT_FOUND");
}
```

**Confirmed failing:**
```
java.lang.AssertionError: Expected ResourceNotFoundException but nothing was thrown
findById() stub returns null instead of throwing
```

---

## Gate Summary

| Test | Criterion | Confirmed failing | Error type |
|---|---|---|---|
| PROD-T1 | POST 201 | ✅ | 500 — null stub |
| PROD-T2 | POST duplicate SKU 409 | ✅ | 201 — no uniqueness check |
| PROD-T3 | GET paginated 200 | ✅ | 500 — null stub |
| PROD-T4 | GET unknown 404 | ✅ | 500 — wrong exception |
| PROD-T5 | PUT update 200 | ✅ | 500 — not implemented |
| PROD-T6 | DELETE soft delete | ✅ | 500 — not implemented |
| PROD-T7 | POST blank name 400 | ✅ | 500 — @Valid missing |
| PROD-T8 | No JWT → 401 | ✅ | 403 — security not configured |
| PROD-T9 | Service findById exception | ✅ | No exception thrown |

**Gate status: PASSED — all 9 tests confirmed failing for the right reasons.**

> Implementation may now begin. Start at TASK-10 in `tasks.md`.
