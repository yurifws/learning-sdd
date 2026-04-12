# audit-trail.md — User Authentication Feature

> **Feature:** User Authentication (Login)
> **Date started:** 2026-04-11
> **Date completed:** 2026-04-14
> **Status:** ✅ All requirements done

---

## Audit Trail

| Req ID | Scenario | Type | Test | Verification Command | Status | PR |
|---|---|---|---|---|---|---|
| AUTH-001 | Successful login returns JWT | Happy | `test_login_returns_jwt` | `pytest tests/auth/test_login.py::test_login_returns_jwt -v` | ✅ | #17 |
| AUTH-001 | Token expires in exactly 3600 seconds | Happy | `test_token_expiry_is_one_hour` | `pytest tests/auth/test_login.py::test_token_expiry_is_one_hour -v` | ✅ | #17 |
| AUTH-001 | Refresh token stored in httpOnly cookie | Happy | `test_refresh_token_in_cookie` | `pytest tests/auth/test_login.py::test_refresh_token_in_cookie -v` | ✅ | #17 |
| AUTH-001 | Login response contains no sensitive data | Happy | `test_response_has_no_sensitive_data` | `pytest tests/auth/test_login.py::test_response_has_no_sensitive_data -v` | ✅ | #17 |
| AUTH-002 | Expiring-today password: login succeeds with warning | Edge | `test_expiring_password_returns_warning_flag` | `pytest tests/auth/test_login.py::test_expiring_password_returns_warning_flag -v` | ✅ | #17 |
| AUTH-002 | Concurrent logins: one session created | Edge | `test_concurrent_logins_single_session` | `pytest tests/auth/test_login.py::test_concurrent_logins_single_session -v` | ✅ | #17 |
| AUTH-002 | Email is case-insensitive | Edge | `test_email_case_insensitive` | `pytest tests/auth/test_login.py::test_email_case_insensitive -v` | ✅ | #17 |
| AUTH-002 | Password at max length (128 chars) accepted | Edge | `test_max_length_password` | `pytest tests/auth/test_login.py::test_max_length_password -v` | ✅ | #17 |
| AUTH-003 | Wrong password: 401, no email enumeration | Error | `test_wrong_password_returns_401` | `pytest tests/auth/test_login.py::test_wrong_password_returns_401 -v` | ✅ | #17 |
| AUTH-003 | Non-existent email: same response as wrong password | Error | `test_nonexistent_email_same_response` | `pytest tests/auth/test_login.py::test_nonexistent_email_same_response -v` | ✅ | #17 |
| AUTH-003 | Locked account: 403 even with correct credentials | Error | `test_locked_account_returns_403` | `pytest tests/auth/test_login.py::test_locked_account_returns_403 -v` | ✅ | #17 |
| AUTH-003 | Service unavailable: 503, no internal details | Error | `test_service_unavailable_returns_503` | `pytest tests/auth/test_login.py::test_service_unavailable_returns_503 -v` | ✅ | #17 |
| AUTH-003 | Missing fields: 400 with specific validation errors | Error | `test_missing_fields_returns_400` | `pytest tests/auth/test_login.py::test_missing_fields_returns_400 -v` | ✅ | #17 |
| AUTH-004 | Any valid password can be stored and verified (PBT) | Property | `test_any_valid_password_can_be_stored_and_verified` | `pytest tests/auth/test_login_properties.py -v` | ✅ | #18 |
| AUTH-004 | Wrong password always 401, never token (PBT) | Property | `test_wrong_password_always_returns_401_never_token` | `pytest tests/auth/test_login_properties.py -v` | ✅ | #18 |
| AUTH-004 | Failed attempt count always increments by 1 (PBT) | Property | `test_failed_attempt_count_always_increments_by_one` | `pytest tests/auth/test_login_properties.py -v` | ✅ | #18 |
| AUTH-004 | Response timing consistent — no email enumeration (PBT) | Property | `test_timing_consistent_between_existing_and_nonexistent_email` | `pytest tests/auth/test_login_properties.py -v` | ✅ | #18 |

---

## Full Verification

```bash
# Run everything — must exit 0
pytest tests/auth/ -v

# Expected output
tests/auth/test_login.py::test_login_returns_jwt PASSED
tests/auth/test_login.py::test_token_expiry_is_one_hour PASSED
tests/auth/test_login.py::test_refresh_token_in_cookie PASSED
tests/auth/test_login.py::test_response_has_no_sensitive_data PASSED
tests/auth/test_login.py::test_expiring_password_returns_warning_flag PASSED
tests/auth/test_login.py::test_concurrent_logins_single_session PASSED
tests/auth/test_login.py::test_email_case_insensitive PASSED
tests/auth/test_login.py::test_max_length_password PASSED
tests/auth/test_login.py::test_wrong_password_returns_401 PASSED
tests/auth/test_login.py::test_nonexistent_email_same_response PASSED
tests/auth/test_login.py::test_locked_account_returns_403 PASSED
tests/auth/test_login.py::test_service_unavailable_returns_503 PASSED
tests/auth/test_login.py::test_missing_fields_returns_400 PASSED
tests/auth/test_login_properties.py::test_any_valid_password_can_be_stored_and_verified PASSED
tests/auth/test_login_properties.py::test_wrong_password_always_returns_401_never_token PASSED
tests/auth/test_login_properties.py::test_failed_attempt_count_always_increments_by_one PASSED
tests/auth/test_login_properties.py::test_timing_consistent_between_existing_and_nonexistent_email PASSED

17 passed in 8.43s
```

---

## Feature Done

- ✅ 17 scenarios written before implementation
- ✅ 17 tests confirmed failing before implementation (gate opened 2026-04-11)
- ✅ 13 example-based tests passing (PR #17 merged 2026-04-14)
- ✅ 4 property-based tests passing, 500+ generated inputs each (PR #18 merged 2026-04-14)
- ✅ Audit trail complete — every requirement traces to a scenario, a test, and a PR
