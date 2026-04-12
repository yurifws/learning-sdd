# TEST_FIRST_GATE.md

> In spec-driven development, test-first is governance — not preference.
> No implementation begins until this gate is passed.

---

## The Rule

```
Spec → EARS requirements → Scenarios (G/W/T) → Tests (confirmed failing) → Implementation
```

The gate sits between scenarios and implementation. It ensures every requirement has a test, every test is meaningful (it fails), and the definition of done is objective before a single line of code is written.

---

## The Gate Checklist

### 1. Every EARS clause maps to at least one test

| EARS pattern | Required scenarios |
|---|---|
| Ubiquitous (`SHALL`) | At least one happy path confirming the rule holds |
| Event-driven (`WHEN`) | Happy path + at least one error case |
| Conditional (`IF/THEN`) | The `THEN` branch + the `ELSE`/default branch |
| Unwanted (`SHALL NOT`) | At least one test asserting the forbidden behavior is absent |

- [ ] Every `SHALL` has a test
- [ ] Every `SHALL NOT` has a negative assertion

---

### 2. Happy path, edge cases, and error cases are all covered

- [ ] At least one scenario per EARS clause covers the ideal flow
- [ ] Boundary values from the EARS clause are tested (max length, null, zero, empty)
- [ ] Every error path described in the EARS spec has a test

---

### 3. Acceptance criteria map 1:1 to tests

Check the requirements.md acceptance criteria list against the test file:

- [ ] Every checkbox in the acceptance criteria has a corresponding test function
- [ ] No test exists without a corresponding acceptance criterion

---

### 4. Verification commands are in tasks.md

Each task in `tasks.md` must have a runnable `Acceptance:` block with a specific command:

```
**Acceptance:**
./mvnw test -Dtest=LinkResolutionIT -v
→ All tests pass.
```

"Run the tests" is not a verification command.

- [ ] Every task has a specific, runnable `Acceptance:` command
- [ ] Commands are runnable in CI without modification

---

### 5. Tests confirmed failing ✗

Run the full test suite against the current codebase (no implementation yet):

- [ ] Every new test fails with a **semantic** error (expected 200, got 404)
- [ ] No new test fails with a **setup** error (ImportError, NullPointerException in setup)
- [ ] No new test passes before implementation

A test that passes before the code exists is not testing the behavior — it is a false green.

---

## Gate Sign-Off Template

Add this block to the relevant task in `tasks.md` when the gate is cleared:

```markdown
**Test-First Gate: ✅ OPEN**
- EARS clauses covered: [list]
- Tests confirmed failing: [count] tests, all semantic failures
- Verification command: [command]
- Date: YYYY-MM-DD
```

---

## In the SDD Workflow

The gate fits between the spec step and the implementation step:

```
EARS requirements (requirements.md)
         ↓
Scenarios + tests written (from acceptance criteria)
         ↓
TEST-FIRST GATE ← here
         ↓
Tasks fed to AI one at a time (tasks.md)
         ↓
Verify: run the command from tasks.md
```

The AI's definition of done is the gate itself: make the tests pass, nothing more.
