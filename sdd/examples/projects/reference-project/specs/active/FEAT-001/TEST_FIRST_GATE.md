# Test-First Gate — FEAT-001: Send Notification

> **Rule:** Every test listed here must exist AND be failing (red) before Phase 1 implementation begins.  
> A passing test before the code exists is a false green — the gate is broken.  
> **Status:** ✅ All 11 tests confirmed failing | **Date:** 2026-01-16 | **Verified by:** Platform Team

---

## Confirmation Command

```bash
pytest tests/test_notifications.py -v
# Expected: 11 FAILED, 0 passed
```

---

## Required Tests

| # | Test name | Scenario | EARS ref |
|---|-----------|----------|----------|
| T-01 | `test_send_notification_returns_202` | SC-01 happy path | EARS-01 |
| T-02 | `test_send_notification_persists_record` | SC-01 DB record | EARS-01 |
| T-03 | `test_delivery_success_updates_status` | SC-01 delivery | EARS-02, EARS-03 |
| T-04 | `test_get_notification_returns_status` | SC-02 | EARS-04 |
| T-05 | `test_retry_scheduled_after_failure` | SC-03 | EARS-05, EARS-06 |
| T-06 | `test_notification_fails_after_max_retries` | SC-04 | EARS-07 |
| T-07 | `test_missing_api_key_returns_401` | SC-05 | EARS-08 |
| T-08 | `test_missing_field_returns_422` | SC-06 | EARS-09 |
| T-09 | `test_unknown_template_returns_422` | SC-07 | EARS-10 |
| T-10 | `test_user_service_unavailable_returns_503` | SC-08 | EARS-11 |
| T-11 | `test_unknown_notification_id_returns_404` | SC-09 | EARS-12 |

---

## Security Tests

| # | Test name | Scenario | Ref |
|---|-----------|----------|-----|
| S-01 | `test_no_pii_in_log_output` | SC-10 | Constitution § Security Gate |
| S-02 | `test_error_response_has_no_traceback` | SC-11 | Constitution § Security Gate |

---

## Pre-Gate Checklist

- [x] All 11 functional tests written and failing red
- [x] Both security tests written and failing red
- [x] Every EARS clause from `requirements.md` maps to at least one test
- [x] Every scenario in `scenarios.md` maps to at least one test
- [x] No test is trivially passing (e.g., `assert True`)
- [x] Test file is committed on the spec branch before implementation begins

---

## Failure Log (pre-implementation)

```
FAILED tests/test_notifications.py::test_send_notification_returns_202 - ImportError: cannot import name 'app' from 'app.main'
FAILED tests/test_notifications.py::test_send_notification_persists_record - ImportError
FAILED tests/test_notifications.py::test_delivery_success_updates_status - ImportError
FAILED tests/test_notifications.py::test_get_notification_returns_status - ImportError
FAILED tests/test_notifications.py::test_retry_scheduled_after_failure - ImportError
FAILED tests/test_notifications.py::test_notification_fails_after_max_retries - ImportError
FAILED tests/test_notifications.py::test_missing_api_key_returns_401 - ImportError
FAILED tests/test_notifications.py::test_missing_field_returns_422 - ImportError
FAILED tests/test_notifications.py::test_unknown_template_returns_422 - ImportError
FAILED tests/test_notifications.py::test_user_service_unavailable_returns_503 - ImportError
FAILED tests/test_notifications.py::test_unknown_notification_id_returns_404 - ImportError
FAILED tests/test_notifications.py::test_no_pii_in_log_output - ImportError
FAILED tests/test_notifications.py::test_error_response_has_no_traceback - ImportError

13 failed in 0.12s
```

> ✅ Gate confirmed open. Implementation may begin at T-01.
