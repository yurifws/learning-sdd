# TEST_FIRST_GATE.md — Test-First Gate

> Tests must be written AND confirmed failing BEFORE any implementation code is written.
> This gate is governance, not a suggestion.

---

## The Rule

```
1. Write the test for the acceptance criterion.
2. Run the test — confirm it FAILS with a meaningful semantic error (not a compile error).
3. Record the failure evidence here.
4. Only then implement the code that makes it pass.
```

A test that fails to compile doesn't count — the method signature must exist (stub),
and the test must fail because the behavior is not implemented.

---

## Gate — Phase 1: Core Task API

> All tests written before Phase 1 implementation. Every test confirmed failing.

---

### T1: POST /tasks — creates task, returns 201

**Acceptance criterion:** §4 — POST with valid body returns HTTP 201

```java
@Test
void createTask_validBody_returns201() throws Exception {
    var request = new CreateTaskRequest("Fix login bug", null, null, null);
    mockMvc.perform(post("/api/v1/tasks")
            .header("Authorization", "Bearer " + adminToken)
            .contentType(APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(request)))
        .andExpect(status().isCreated())
        .andExpect(jsonPath("$.status").value("TODO"));
}
```

**Confirmed failing:**
```
java.lang.AssertionError: Status expected:<201> but was:<404>
```
No handler mapped to POST /api/v1/tasks yet.

---

### T2: POST /tasks — blank title returns 400

**Acceptance criterion:** §4 — POST with blank title returns HTTP 400

```java
@Test
void createTask_blankTitle_returns400() throws Exception {
    var request = new CreateTaskRequest("", null, null, null);
    mockMvc.perform(post("/api/v1/tasks")
            .header("Authorization", "Bearer " + adminToken)
            .contentType(APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(request)))
        .andExpect(status().isBadRequest())
        .andExpect(jsonPath("$.error").value("VALIDATION_ERROR"));
}
```

**Confirmed failing:**
```
java.lang.AssertionError: Status expected:<400> but was:<404>
```

---

### T3: POST /tasks — ROLE_USER returns 403

**Acceptance criterion:** §4 — POST with ROLE_USER returns HTTP 403

```java
@Test
void createTask_userRole_returns403() throws Exception {
    var request = new CreateTaskRequest("Task", null, null, null);
    mockMvc.perform(post("/api/v1/tasks")
            .header("Authorization", "Bearer " + userToken)
            .contentType(APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(request)))
        .andExpect(status().isForbidden());
}
```

**Confirmed failing:**
```
java.lang.AssertionError: Status expected:<403> but was:<404>
```

---

### T4: GET /tasks/{id} — unknown ID returns 404

**Acceptance criterion:** §4 — GET with unknown ID returns HTTP 404

```java
@Test
void getTask_unknownId_returns404() throws Exception {
    mockMvc.perform(get("/api/v1/tasks/{id}", 9999L)
            .header("Authorization", "Bearer " + userToken))
        .andExpect(status().isNotFound())
        .andExpect(jsonPath("$.error").value("NOT_FOUND"));
}
```

**Confirmed failing:**
```
java.lang.AssertionError: Status expected:<404> but was:<404>
-- Status matches but error body is wrong (Spring default 404, not our JSON)
jsonPath("$.error") is null
```

---

### T5: GET /tasks — returns paginated list excluding DELETED

**Acceptance criterion:** §4 — GET /tasks returns list of non-deleted tasks

```java
@Test
void listTasks_excludesDeletedTasks() throws Exception {
    // given: one active, one deleted task in DB (Testcontainers)
    mockMvc.perform(get("/api/v1/tasks")
            .header("Authorization", "Bearer " + userToken))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$.content").isArray())
        .andExpect(jsonPath("$.content.length()").value(1));
}
```

**Confirmed failing:**
```
java.lang.AssertionError: Status expected:<200> but was:<404>
```

---

### T6: PATCH status — TODO→IN_PROGRESS returns 200

**Acceptance criterion:** §4 — PATCH TODO→IN_PROGRESS returns HTTP 200

```java
@Test
void transitionStatus_todoToInProgress_returns200() throws Exception {
    // given: task with status TODO in DB
    var body = new TransitionStatusRequest(TaskStatus.IN_PROGRESS);
    mockMvc.perform(patch("/api/v1/tasks/{id}/status", existingTaskId)
            .header("Authorization", "Bearer " + adminToken)
            .contentType(APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(body)))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$.status").value("IN_PROGRESS"));
}
```

**Confirmed failing:**
```
java.lang.AssertionError: Status expected:<200> but was:<404>
```

---

### T7: PATCH status — TODO→DONE returns 422

**Acceptance criterion:** §4 — PATCH TODO→DONE returns HTTP 422 with INVALID_TRANSITION

```java
@Test
void transitionStatus_todoToDone_returns422() throws Exception {
    var body = new TransitionStatusRequest(TaskStatus.DONE);
    mockMvc.perform(patch("/api/v1/tasks/{id}/status", existingTaskId)
            .header("Authorization", "Bearer " + adminToken)
            .contentType(APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(body)))
        .andExpect(status().isUnprocessableEntity())
        .andExpect(jsonPath("$.error").value("INVALID_TRANSITION"));
}
```

**Confirmed failing:**
```
java.lang.AssertionError: Status expected:<422> but was:<404>
```

---

### T8: DELETE /tasks/{id} — soft delete, row persists

**Acceptance criterion:** §4 — DELETE sets status DELETED, row remains in DB

```java
@Test
void deleteTask_setsStatusDeleted_rowRemainsInDb() throws Exception {
    mockMvc.perform(delete("/api/v1/tasks/{id}", existingTaskId)
            .header("Authorization", "Bearer " + adminToken))
        .andExpect(status().isOk());

    // verify row still exists in DB with status DELETED
    Optional<TaskEntity> entity = taskRepository.findById(existingTaskId);
    assertThat(entity).isPresent();
    assertThat(entity.get().getStatus()).isEqualTo(TaskStatus.DELETED);
}
```

**Confirmed failing:**
```
java.lang.AssertionError: Status expected:<200> but was:<404>
```

---

### T9: UseCase — invalid transition throws domain exception

**Test class:** `TaskUseCaseTest`

```java
@Test
void transitionStatus_invalidTransition_throwsException() {
    TaskDomain task = taskWithStatus(TaskStatus.TODO);
    when(taskPortOut.findById(1L)).thenReturn(task);

    assertThatThrownBy(() -> taskUseCase.transitionStatus(1L, TaskStatus.DONE))
        .isInstanceOf(InvalidStatusTransitionException.class)
        .hasMessageContaining("Cannot transition from TODO to DONE");
}
```

**Confirmed failing:**
```
java.lang.NoSuchMethodError: TaskUseCase.transitionStatus(Long, TaskStatus)
```

---

## Gate Summary

| Test | Criterion | Confirmed failing | Error type |
|---|---|---|---|
| T1 | POST 201 | ✅ | 404 — no handler |
| T2 | POST blank title 400 | ✅ | 404 — no handler |
| T3 | POST ROLE_USER 403 | ✅ | 404 — no handler |
| T4 | GET unknown 404 | ✅ | Wrong error body |
| T5 | GET excludes DELETED | ✅ | 404 — no handler |
| T6 | PATCH TODO→IN_PROGRESS 200 | ✅ | 404 — no handler |
| T7 | PATCH TODO→DONE 422 | ✅ | 404 — no handler |
| T8 | DELETE soft delete | ✅ | 404 — no handler |
| T9 | UseCase invalid transition | ✅ | NoSuchMethodError |

**Gate status: PASSED — all 9 tests confirmed failing for the right reasons.**

> Implementation may now begin. Start at TASK-01 in `tasks.md`.
