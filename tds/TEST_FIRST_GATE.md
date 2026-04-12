# TEST_FIRST_GATE.md

> Test-first is not a developer preference. It is a governance policy.
> By requiring tests before implementation, you catch flawed assumptions when they are cheapest to fix — at the start.

---

## Why the Gate Exists

Without a test-first gate, this is what happens:
1. Engineer reads the spec
2. Engineer writes code based on their interpretation
3. Engineer writes tests to match the code they already wrote
4. Tests pass — because they were written to match existing code, not the original intent
5. The spec drift is invisible until QA or production

The test-first gate inverts this. Tests are written from the spec. Then they must **fail**. Only then does implementation begin. A test that passes before the code exists is not testing anything.

---

## The Gate Checklist

Run this checklist for every requirement before implementation starts.
All items must pass. No exceptions.

### Step 1 — Requirement coverage

- [ ] Every requirement has at least one scenario (Given/When/Then)
- [ ] Every scenario has a test file and a test function (skeleton, at minimum)
- [ ] Happy path, edge cases, and error cases are all represented

**Fail condition:** Any requirement with no corresponding test function → stop. Write the test first.

---

### Step 2 — Negative case coverage

- [ ] Every error case scenario has a test with an explicit negative assertion
- [ ] Every `does NOT` clause has a test that asserts the forbidden outcome is absent
- [ ] Security-sensitive paths (auth, PII, deletion) have at least one error case test

**Fail condition:** Error scenario exists in the spec but no corresponding test → stop.

---

### Step 3 — Verification commands declared

- [ ] Every test has a runnable verification command documented in the task
- [ ] The command is specific enough to run in CI without modification

**Examples of acceptable verification commands:**
```bash
pytest tests/auth/test_login.py -v
npm test -- --testPathPattern="auth/login"
./mvnw test -Dtest=LoginControllerIT
```

**Fail condition:** "Run the tests" is not a verification command. A specific command must exist.

---

### Step 4 — Tests confirmed failing ✗

This is the most important step.

- [ ] Run every test written for this requirement
- [ ] Confirm every test **fails** with a meaningful error (not a setup error)
- [ ] Record the failure output

**Why this matters:**
- A test that passes before implementation means it is not actually testing the behavior
- A setup error (import failure, missing fixture) means the test is broken, not meaningful
- Only a semantic failure — "expected 200 but got 404" — proves the test is real

**Acceptable failure:**
```
FAILED tests/auth/test_login.py::test_login_returns_jwt - AssertionError: expected 200, got 404
```

**Not acceptable (test is broken, not meaningful):**
```
ERROR tests/auth/test_login.py::test_login_returns_jwt - ImportError: cannot import 'login_service'
```

**Fail condition:** Any test passes before implementation → the test does not prove the behavior. Rewrite it.

---

### Step 5 — Gate sign-off

| Item | Status | Notes |
|---|---|---|
| All requirements covered | ☐ Pass / ☐ Fail | |
| All negative cases covered | ☐ Pass / ☐ Fail | |
| Verification commands declared | ☐ Pass / ☐ Fail | |
| All tests confirmed failing | ☐ Pass / ☐ Fail | |
| **Gate status** | **☐ OPEN / ☐ CLOSED** | |

Gate is **OPEN** only when all four items pass. Implementation may not begin until the gate is open.

---

## Enforcing the Gate

The gate is most effective when it is automated, not manual.

### In CI

Add a gate step to your pipeline that runs before the implementation PR is merged:

```yaml
# .github/workflows/test-first-gate.yml
jobs:
  gate:
    steps:
      - name: All tests must exist before implementation
        run: |
          # Count test functions in test files
          TEST_COUNT=$(grep -r "def test_\|it(" tests/ | wc -l)
          if [ "$TEST_COUNT" -eq 0 ]; then
            echo "GATE BLOCKED: No test functions found."
            exit 1
          fi

      - name: Run tests — expect failures
        run: pytest tests/ --tb=short
        # This step is informational — failures are expected at this point
        continue-on-error: true
```

### In Code Review

The SPEC PR (containing only scenarios and tests, no implementation) is reviewed before the IMPL PR is opened:

1. Open **SPEC PR**: scenarios + test skeletons + gate checklist
2. Team reviews: are all cases covered? Is the gate checklist complete?
3. Merge SPEC PR
4. Open **IMPL PR**: implementation code that makes tests pass
5. Team reviews: does the code match the scenarios? Do all tests pass?

This makes the gate a natural part of the review process.

---

## Red Flags

These patterns indicate the gate was skipped or bypassed:

| Red Flag | What it usually means |
|---|---|
| Tests added in the same commit as implementation | Test written after code — drift risk |
| All tests pass immediately on first run | Tests written to match code, not spec |
| Test file has only one test per file | Edge cases and error cases were skipped |
| Test names are method names, not behaviors | Tests describe code, not requirements |
| No test for the error case | The error case wasn't thought through |

---

## The Payoff

When the gate is enforced consistently:

- Flawed assumptions are caught in the spec, not in production
- AI agents have an unambiguous definition of done — the tests
- Code reviews become faster — "do the tests pass?" is the primary question
- Onboarding is faster — new engineers understand a feature by reading its tests
- Regressions are caught immediately — any change that breaks a spec-derived test is visible

Done is no longer an opinion. It is a verifiable fact.
