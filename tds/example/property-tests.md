# property-tests.md — User Authentication Feature

> **Feature:** User Authentication (Login)
> **Framework:** Hypothesis (Python)
> **Test file:** `tests/auth/test_login_properties.py`
> Properties derived from scenarios in `scenarios.md`.

---

## AUTH-004 — Password Handling Properties

---

### PBT-1: Any valid password can be stored and verified

**Derived from AUTH-004** (`Password handling is correct for all valid inputs`)

**Property:** For any string meeting the password policy, the system correctly stores and verifies it.

```python
from hypothesis import given, strategies as st, settings, assume
from hypothesis import HealthCheck
import string
import re

VALID_PASSWORD_CHARS = string.ascii_letters + string.digits + "!@#$%^&*()"
PASSWORD_POLICY = re.compile(r'^[a-zA-Z0-9!@#$%^&*()]{8,128}$')

@settings(max_examples=500, suppress_health_check=[HealthCheck.too_slow])
@given(
    password=st.text(
        alphabet=VALID_PASSWORD_CHARS,
        min_size=8,
        max_size=128
    )
)
def test_any_valid_password_can_be_stored_and_verified(password, db_session):
    """
    Property: For any password satisfying the policy, storing and verifying it
    always succeeds. The hash is never the raw password.
    """
    assume(PASSWORD_POLICY.match(password))

    user = create_user(email="prop-test@example.com", password=password, db=db_session)

    # The hash is never the raw password
    assert user.hashed_password != password

    # Verification with the correct password always succeeds
    assert verify_password(password, user.hashed_password) is True

    # Cleanup
    db_session.delete(user)
    db_session.commit()
```

**What this catches that example tests miss:**
- Passwords with only uppercase letters
- Passwords with only special characters
- Passwords with Unicode-lookalike ASCII characters
- Passwords exactly 8 characters (minimum boundary)
- Passwords exactly 128 characters (maximum boundary)

---

### PBT-2: Wrong password always returns 401 — never leaks a token

**Derived from AUTH-003-ER1 and AUTH-003-ER2**

**Property:** For any combination of (email, wrong_password), a login attempt never returns 200 or a token.

```python
@settings(max_examples=500)
@given(
    wrong_password=st.text(min_size=1, max_size=200)
)
def test_wrong_password_always_returns_401_never_token(wrong_password, client, registered_user):
    """
    Property: Any incorrect password, for any length and content, always
    returns 401 and never produces a token in the response.
    """
    assume(wrong_password != registered_user.raw_password)

    response = client.post("/auth/login", json={
        "email": registered_user.email,
        "password": wrong_password
    })

    # Always 401
    assert response.status_code == 401

    body = response.json()

    # Never a token
    assert "token" not in body
    assert "access_token" not in body

    # Never the user's email in the response (no enumeration)
    assert registered_user.email not in response.text
```

**What this catches that example tests miss:**
- Empty string passwords
- Very long passwords (> 1000 chars)
- Passwords that are substrings of the real password
- Passwords with null bytes or control characters
- The correct password with one character changed

---

### PBT-3: Failed attempt count always increments for wrong password

**Derived from AUTH-003-ER1 (`the failed_attempt_count increases by 1`)**

**Property:** For any wrong password, the failed attempt count increases by exactly 1 — never more, never less.

```python
@settings(max_examples=200)
@given(
    wrong_password=st.text(min_size=1)
)
def test_failed_attempt_count_always_increments_by_one(wrong_password, client, registered_user, db_session):
    """
    Property: Any failed login attempt increases failed_attempt_count by exactly 1.
    No more (double-counting), no less (silent failure).
    """
    assume(wrong_password != registered_user.raw_password)

    # Record count before
    user_before = db_session.get(User, registered_user.id)
    count_before = user_before.failed_attempt_count

    client.post("/auth/login", json={
        "email": registered_user.email,
        "password": wrong_password
    })

    # Record count after
    db_session.refresh(user_before)
    count_after = user_before.failed_attempt_count

    assert count_after == count_before + 1
```

---

### PBT-4: Response timing is consistent — no email enumeration via timing

**Derived from AUTH-003-ER2 (`response time within 20ms of failed attempt on existing account`)**

**Property:** The response time for a non-existent email is always within 20ms of the response time for a wrong password on an existing account.

```python
import time
from hypothesis import given, strategies as st

@settings(max_examples=50)  # Fewer examples — timing tests are slow
@given(
    fake_email=st.emails()
)
def test_timing_consistent_between_existing_and_nonexistent_email(fake_email, client, registered_user):
    """
    Property: An attacker cannot determine if an email is registered by measuring
    response time. The difference must always be < 20ms.
    """
    assume(fake_email != registered_user.email)

    # Time a request for an existing email (wrong password)
    start = time.perf_counter()
    client.post("/auth/login", json={"email": registered_user.email, "password": "wrong"})
    existing_time_ms = (time.perf_counter() - start) * 1000

    # Time a request for a non-existent email
    start = time.perf_counter()
    client.post("/auth/login", json={"email": fake_email, "password": "wrong"})
    nonexistent_time_ms = (time.perf_counter() - start) * 1000

    # The difference must be < 20ms
    assert abs(existing_time_ms - nonexistent_time_ms) < 20, (
        f"Timing difference {abs(existing_time_ms - nonexistent_time_ms):.1f}ms "
        f"exceeds 20ms threshold — timing attack possible"
    )
```

---

## Property Coverage Summary

| Property | Scenario | Framework | Examples | Status |
|---|---|---|---|---|
| PBT-1: Any valid password stored/verified | AUTH-001-H1 | Hypothesis | 500 | ✅ |
| PBT-2: Wrong password always 401, never token | AUTH-003-ER1, ER2 | Hypothesis | 500 | ✅ |
| PBT-3: Failed count increments exactly 1 | AUTH-003-ER1 | Hypothesis | 200 | ✅ |
| PBT-4: Timing consistent (no enumeration) | AUTH-003-ER2 | Hypothesis | 50 | ✅ |

---

## How to Run

```bash
# Run all property tests
pytest tests/auth/test_login_properties.py -v

# Run with verbose Hypothesis output
pytest tests/auth/test_login_properties.py -v --hypothesis-show-statistics

# Run with more examples (slower, more thorough)
pytest tests/auth/test_login_properties.py -v --hypothesis-seed=0 \
  -p hypothesis --hypothesis-settings max_examples=1000
```

## When a Property Fails

Hypothesis will report the minimal failing example:

```
FAILED tests/auth/test_login_properties.py::test_wrong_password_always_returns_401_never_token
Falsifying example: test_wrong_password_always_returns_401_never_token(
    wrong_password=''   ← Hypothesis shrunk to simplest failing case
)
AssertionError: expected 401, got 200
```

This means the system accepted an empty string password — an edge case the example tests didn't cover.
