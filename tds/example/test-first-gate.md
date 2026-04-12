# test-first-gate.md — User Authentication Feature

> **Feature:** User Authentication (Login)
> **Gate run by:** Platform Team
> **Date:** 2026-04-11
> **Status:** ✅ OPEN — implementation may begin

---

## Step 1 — Requirement Coverage

| Requirement | Scenarios written | Tests written | All types covered? |
|---|---|---|---|
| AUTH-001 | H1, H2, H3 | ✅ 3 tests | ✅ Happy path |
| AUTH-002 | E1, E2, E3, E4 | ✅ 4 tests | ✅ Edge cases |
| AUTH-003 | ER1, ER2, ER3, ER4, ER5 | ✅ 5 tests | ✅ Error cases |
| AUTH-004 | PBT-1, PBT-2 | ✅ 2 properties | ✅ Properties |

**Result: ✅ PASS** — All requirements have scenarios and test functions.

---

## Step 2 — Negative Case Coverage

| Scenario | Negative assertion present? |
|---|---|
| AUTH-003-ER1: Wrong password | ✅ `does NOT reveal email exists` → `assert "user@example.com" not in response.text` |
| AUTH-003-ER2: Non-existent email | ✅ Response identical to ER1 (timing + body) |
| AUTH-003-ER3: Locked account | ✅ `does NOT contain token` → `assert "token" not in response.json()` |
| AUTH-003-ER4: Service unavailable | ✅ `does NOT contain stack trace` → `assert "Traceback" not in response.text` |
| AUTH-001-H3: No sensitive data in response | ✅ `does NOT contain hashed_password` → `assert "hashed_password" not in response.json()` |

**Result: ✅ PASS** — All negative cases have explicit assertions.

---

## Step 3 — Verification Commands Declared

```bash
# All authentication tests
pytest tests/auth/ -v

# Happy path only
pytest tests/auth/test_login.py -v -k "happy"

# Edge cases only
pytest tests/auth/test_login.py -v -k "edge"

# Error cases only
pytest tests/auth/test_login.py -v -k "error"

# Property-based tests only
pytest tests/auth/test_login_properties.py -v

# Full suite with coverage report
pytest tests/auth/ -v --cov=app.auth --cov-report=term-missing
```

**Result: ✅ PASS** — Specific, runnable commands documented for every category.

---

## Step 4 — Tests Confirmed Failing ✗

All tests were run against the empty implementation (no login route exists yet):

```
FAILED tests/auth/test_login.py::test_login_returns_jwt
  AssertionError: expected 200, got 404

FAILED tests/auth/test_login.py::test_token_expiry_is_one_hour
  AssertionError: expected 200, got 404

FAILED tests/auth/test_login.py::test_refresh_token_in_cookie
  AssertionError: expected 200, got 404

FAILED tests/auth/test_login.py::test_expiring_password_returns_warning_flag
  AssertionError: expected 200, got 404

FAILED tests/auth/test_login.py::test_concurrent_logins_single_session
  AssertionError: expected 200, got 404

FAILED tests/auth/test_login.py::test_email_case_insensitive
  AssertionError: expected 200, got 404

FAILED tests/auth/test_login.py::test_max_length_password
  AssertionError: expected 200, got 404

FAILED tests/auth/test_login.py::test_wrong_password_returns_401
  AssertionError: expected 401, got 404

FAILED tests/auth/test_login.py::test_nonexistent_email_same_response
  AssertionError: expected 401, got 404

FAILED tests/auth/test_login.py::test_locked_account_returns_403
  AssertionError: expected 403, got 404

FAILED tests/auth/test_login.py::test_service_unavailable_returns_503
  AssertionError: expected 503, got 404

FAILED tests/auth/test_login.py::test_missing_fields_returns_400
  AssertionError: expected 400, got 404

FAILED tests/auth/test_login_properties.py::test_any_valid_password_verifiable
  hypothesis.errors.Unsatisfiable: Unable to satisfy assume() — no route exists

FAILED tests/auth/test_login_properties.py::test_wrong_password_always_401
  AssertionError: expected 401, got 404

14 failed, 0 passed
```

All 14 failures are semantic (expected status != received status), not setup errors. The safety net is working.

**Result: ✅ PASS** — All tests confirmed failing with meaningful errors before implementation.

---

## Gate Sign-Off

| Step | Result |
|---|---|
| All requirements covered | ✅ PASS |
| All negative cases covered | ✅ PASS |
| Verification commands declared | ✅ PASS |
| All tests confirmed failing | ✅ PASS |
| **Gate status** | **✅ OPEN** |

**Implementation may begin.**

The definition of done: `pytest tests/auth/ -v` exits with 0 failures.
