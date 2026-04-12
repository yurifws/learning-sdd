# DEFINITION_OF_DONE.md

> A requirement is done when — and only when — every scenario derived from it has a passing, automated test.
> Done is a verifiable fact, not an opinion.

---

## The Rule

```
Requirement → Scenario (G/W/T) → Test → ✅ Passing

If any link in this chain is missing, the requirement is NOT done.
```

No exceptions. A requirement with scenarios but no tests is not done. A test that passes but has no linked scenario is floating — it proves nothing about the spec.

---

## The Audit Trail Table

Every requirement gets a row. Every row links requirement to scenario to test to PR.

This table is the living, auditable proof that the spec was implemented correctly.

### Template

| Req ID | Scenario | Type | Test | Verification Command | Status | PR |
|---|---|---|---|---|---|---|
| REQ-001 | Happy path description | Happy | `tests/path/test_file.py::test_name` | `pytest tests/path/test_file.py -v` | ✅ | #42 |
| REQ-001 | Edge case description | Edge | `tests/path/test_file.py::test_edge` | `pytest tests/path/test_file.py -v` | ✅ | #42 |
| REQ-001 | Error case description | Error | `tests/path/test_file.py::test_error` | `pytest tests/path/test_file.py -v` | ✅ | #42 |
| REQ-002 | ... | ... | ... | ... | 🔄 | — |

### Status values

| Symbol | Meaning |
|---|---|
| ⬜ | Scenario written, test not yet written |
| 🔴 | Test written, confirmed failing (gate open — ready for implementation) |
| 🔄 | Implementation in progress |
| ✅ | Test passing, PR merged |
| ❌ | Test failing after implementation (regression or broken implementation) |

---

## Filled Example: User Authentication Feature

| Req ID | Scenario | Type | Test | Verification Command | Status | PR |
|---|---|---|---|---|---|---|
| AUTH-001 | Valid credentials return JWT | Happy | `test_login_returns_jwt` | `pytest tests/auth/ -v` | ✅ | #17 |
| AUTH-001 | Token expires in 1 hour | Happy | `test_token_expiry_is_one_hour` | `pytest tests/auth/ -v` | ✅ | #17 |
| AUTH-001 | Refresh token stored in httpOnly cookie | Happy | `test_refresh_token_in_cookie` | `pytest tests/auth/ -v` | ✅ | #17 |
| AUTH-002 | Password expiring today — login succeeds with warning | Edge | `test_expiring_password_returns_warning_flag` | `pytest tests/auth/ -v` | ✅ | #17 |
| AUTH-002 | Concurrent login requests — one session created | Edge | `test_concurrent_logins_single_session` | `pytest tests/auth/ -v` | ✅ | #17 |
| AUTH-003 | Wrong password — 401, no email enumeration | Error | `test_wrong_password_returns_401` | `pytest tests/auth/ -v` | ✅ | #17 |
| AUTH-003 | Locked account — 403 even with correct credentials | Error | `test_locked_account_returns_403` | `pytest tests/auth/ -v` | ✅ | #17 |
| AUTH-003 | Auth service unavailable — 503, no internal details | Error | `test_service_unavailable_returns_503` | `pytest tests/auth/ -v` | ✅ | #17 |
| AUTH-004 | Any valid password can be stored and verified (PBT) | Property | `test_any_valid_password_verifiable` | `pytest tests/auth/test_properties.py -v` | ✅ | #18 |
| AUTH-004 | Wrong password always returns 401 (PBT) | Property | `test_wrong_password_always_401` | `pytest tests/auth/test_properties.py -v` | ✅ | #18 |

---

## What "Done" Looks Like in Practice

### For a single requirement

```
AUTH-001 is done when:
  ✅ test_login_returns_jwt passes
  ✅ test_token_expiry_is_one_hour passes
  ✅ test_refresh_token_in_cookie passes
  and all three are linked in the audit trail
```

### For an entire feature

```
Authentication feature is done when:
  ✅ All rows in the audit trail show ✅
  ✅ pytest tests/auth/ exits with 0 failures
  ✅ All PRs are merged
  ✅ Audit trail is archived with the merged branch
```

### For a release

```
Release is done when:
  ✅ All active features show ✅ in their audit trails
  ✅ CI pipeline runs all verification commands and exits clean
  ✅ No audit trail row is missing a linked PR
```

---

## Common Failure Modes

| Failure mode | What it looks like | What it means |
|---|---|---|
| Row with no test | Scenario exists, test column is empty | Scenario was written but never implemented as a test |
| Row with no scenario | Test exists, scenario column is empty | Test was written directly from code — not from spec |
| All rows "In progress" at PR merge | Audit trail not updated after implementation | Process was not followed — go back and verify |
| No error case rows | Only happy path and edge case rows exist | Error handling was not specified or tested |
| Verification command doesn't run | Command fails in CI | Test is broken or command is stale |

---

## Using the Audit Trail With AI Agents

The audit trail is one of the most powerful tools for working with AI agents.

Instead of telling the agent "implement the login feature", you give it:

```
Implement AUTH-001: Valid credentials return JWT.

Scenarios to make pass:
  - test_login_returns_jwt (currently FAILING — expected 200, got 404)
  - test_token_expiry_is_one_hour (currently FAILING — expected exp-iat=3600, got None)
  - test_refresh_token_in_cookie (currently FAILING — expected Set-Cookie header, got none)

Verification command: pytest tests/auth/test_login.py -v

Done when: all three tests show PASSED.
```

The agent has an unambiguous, mechanically verifiable definition of done. There is no room for drift.
