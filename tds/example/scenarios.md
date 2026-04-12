# scenarios.md — User Authentication Feature

> **Feature:** User Authentication (Login)
> **This file:** All scenarios across happy path, edge cases, and error cases.
> Written before any implementation. Tests derived from these scenarios.

---

## Requirements

| ID | Requirement |
|---|---|
| AUTH-001 | A user with valid credentials can log in and receive a JWT access token |
| AUTH-002 | The system handles boundary conditions in login (expiring passwords, concurrent requests) |
| AUTH-003 | The system rejects invalid, unauthorized, or unavailable login attempts safely |
| AUTH-004 | Password handling is correct for all valid inputs (property) |

---

## AUTH-001 — Happy Path Scenarios

### AUTH-001-H1: Successful login returns JWT

```
Given  a registered user with email "user@example.com" and password "SecurePass123!"
When   the user submits POST /auth/login with { "email": "user@example.com", "password": "SecurePass123!" }
Then   the system returns HTTP 200
  And  the response body contains a "token" field with a valid JWT
  And  the JWT payload contains "sub": "user@example.com"
  And  the JWT expires in exactly 3600 seconds (1 hour)
```

### AUTH-001-H2: Refresh token stored securely

```
Given  a registered user with valid credentials
When   the user successfully logs in
Then   the response sets a "refresh_token" cookie
  And  the cookie has the "HttpOnly" flag set
  And  the cookie has the "Secure" flag set
  And  the cookie expires in 30 days
```

### AUTH-001-H3: Login response does not expose sensitive data

```
Given  a registered user with valid credentials
When   the user successfully logs in
Then   the response body contains "token" and "expires_in"
  And  does NOT contain the user's hashed password
  And  does NOT contain the user's raw password
  And  does NOT contain internal user IDs beyond what is in the JWT payload
```

---

## AUTH-002 — Edge Case Scenarios

### AUTH-002-E1: Password expiring today — login succeeds with warning

```
Given  a registered user whose password expires at the end of today (23:59:59 UTC)
  And  the current time is before 23:59:59 UTC today
When   the user submits valid credentials
Then   the system returns HTTP 200 with a valid JWT
  And  the response body contains "password_expires_soon": true
  And  the response body contains "password_expires_at" with today's date
```

### AUTH-002-E2: Concurrent login requests — exactly one session created

```
Given  a registered user
  And  two POST /auth/login requests are sent with identical valid credentials
  And  both requests arrive within 50ms of each other
When   both requests are processed
Then   both responses return HTTP 200 with valid JWTs
  And  exactly one session record exists in the sessions table for this user
  And  both tokens are valid and independently usable
```

### AUTH-002-E3: Email address is case-insensitive

```
Given  a registered user with email "User@Example.COM"
When   the user submits POST /auth/login with email "user@example.com" (lowercase)
Then   the system returns HTTP 200 with a valid JWT
  And  the JWT subject matches the normalized email "user@example.com"
```

### AUTH-002-E4: Password at maximum length (128 characters)

```
Given  a registered user whose password is exactly 128 characters long
When   the user submits the correct 128-character password
Then   the system returns HTTP 200 with a valid JWT
```

---

## AUTH-003 — Error Case Scenarios

### AUTH-003-ER1: Wrong password — 401, no email enumeration

```
Given  a registered user with email "user@example.com"
When   the user submits POST /auth/login with email "user@example.com" and wrong password "WrongPass!"
Then   the system returns HTTP 401
  And  the response body contains error code "INVALID_CREDENTIALS"
  And  the response body does NOT reveal whether the email exists
  And  the response body does NOT contain the word "password" in the error message
  And  the failed_attempt_count for "user@example.com" increases by 1
```

### AUTH-003-ER2: Non-existent email — same response as wrong password

```
Given  no user is registered with email "ghost@example.com"
When   the user submits POST /auth/login with email "ghost@example.com"
Then   the system returns HTTP 401
  And  the response body contains error code "INVALID_CREDENTIALS"
  And  the response body is identical in structure to AUTH-003-ER1
  And  the response time is within 20ms of a failed attempt on an existing account
```

### AUTH-003-ER3: Locked account — 403 even with correct credentials

```
Given  a user whose account has been locked (failed_attempt_count >= 5)
When   the user submits POST /auth/login with the correct password
Then   the system returns HTTP 403
  And  the response body contains error code "ACCOUNT_LOCKED"
  And  the response body does NOT contain a token
  And  the failed_attempt_count is NOT reset
```

### AUTH-003-ER4: Auth service unavailable — 503, no internal details leaked

```
Given  the token signing service is unavailable (simulated timeout)
When   the user submits valid credentials
Then   the system returns HTTP 503
  And  the response body contains error code "SERVICE_UNAVAILABLE"
  And  the response body does NOT contain stack traces, internal hostnames, or service names
  And  no partial session record is created in the database
```

### AUTH-003-ER5: Missing fields — 400 with specific validation errors

```
Given  a POST /auth/login request with no "password" field
When   the request is received
Then   the system returns HTTP 400
  And  the response body contains error code "VALIDATION_ERROR"
  And  the response body specifies "password is required"
  And  no authentication attempt is made
```

---

## Scenario Coverage Summary

| Requirement | Happy | Edge | Error | Property |
|---|---|---|---|---|
| AUTH-001 | H1, H2, H3 | — | — | AUTH-004 |
| AUTH-002 | — | E1, E2, E3, E4 | — | — |
| AUTH-003 | — | — | ER1, ER2, ER3, ER4, ER5 | — |
| AUTH-004 | — | — | — | See `property-tests.md` |

All three scenario types covered. ✅
