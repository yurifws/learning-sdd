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

---

## Property-Based Tests: When to Upgrade

Some EARS clauses describe a rule for *one specific case*. Others describe a truth that must hold for *all valid inputs*. The second type needs a property-based test, not an example test.

**Signals in the EARS clause that demand a property-based test:**

| Signal word / pattern | What it implies | Example |
|---|---|---|
| `any valid` | The rule must hold for the entire input domain | `WHEN any valid amount is transferred...` |
| `always` / `never` | No exceptions permitted — exhaustive checking required | `The system SHALL never expose...` |
| `exactly` | Precise numeric invariant that boundary inputs easily break | `SHALL increment click_count by exactly 1` |
| `SHALL NOT` on a security rule | One counterexample is a breach — generate thousands of inputs | `SHALL NOT return HTTP 200 for unauthenticated requests` |
| Ubiquitous `SHALL` (no trigger) | Applies to all states — verify it survives edge inputs | `The system SHALL require a valid JWT on all endpoints` |

**The decision rule:**

```
Does the EARS clause use "always", "never", "any", or "exactly"?
  YES → write a property-based test (Hypothesis / FastCheck / jqwik)

Does the EARS clause cover one specific scenario?
  YES → write an example-based test (standard unit/integration test)

Both can coexist — properties prove the rule; examples document specific behavior.
```

A property-based test written at this gate becomes the executable proof that the EARS invariant holds. Add it to the verification command in `tasks.md` alongside the standard test suite.

For framework setup, generator patterns, and advanced examples, see [`tds/PROPERTY_BASED_TESTING.md`](../tds/PROPERTY_BASED_TESTING.md).
