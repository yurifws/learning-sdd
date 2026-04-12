# SCENARIO_FORMAT.md

> Scenarios are the bridge between a business requirement and an automated test.
> They are not user stories. They are precise, structured descriptions of behavior — testable by a machine.

---

## The Given / When / Then Format

```
Given  [the initial state of the system]
When   [the action or trigger occurs]
Then   [the expected outcome]
  And  [additional expected outcome]  (optional chain)
```

Every clause does exactly one thing:

| Clause | Responsibility | What it must NOT do |
|---|---|---|
| `Given` | Set up state — users, data, configuration | Describe actions |
| `When` | Describe one trigger — a request, a click, a timer | Set up state |
| `Then` | Assert the observable outcome | Describe how the outcome is produced |

---

## The Three Scenario Types

Every requirement must be covered by all three types. A spec with only happy paths is incomplete.

---

### Type 1 — Happy Path

The ideal flow. Everything is valid. The system does exactly what it's supposed to.

**Template:**
```
Given  a [valid state]
When   [the user performs the expected action]
Then   the system [produces the expected result]
```

**Example (user authentication):**
```
Given  a registered user with valid credentials
When   the user submits the login form
Then   the system returns HTTP 200 with a JWT access token
  And  the token expires in 1 hour
  And  a refresh token is stored in an httpOnly cookie
```

**Purpose:** Establishes the baseline. If the happy path fails, nothing else matters.

---

### Type 2 — Edge Cases

Valid inputs that push against the boundaries. These are the scenarios that break most systems.

**Common edge case categories:**

| Category | Examples |
|---|---|
| Boundary values | Min/max length, zero, negative, empty string |
| Timing | Tokens expiring mid-request, concurrent requests for the same resource |
| Data shape | Unicode in names, special characters in passwords, null vs. empty |
| Idempotency | Submitting the same request twice |
| Scale | First record, last record, exactly at pagination boundary |

**Example (user authentication):**
```
Given  a registered user whose password expires today at 23:59 UTC
When   the user submits the login form at 23:59:59 UTC
Then   the system returns HTTP 200 with a valid token
  And  includes a "password_expires_soon" flag in the response body

Given  two simultaneous login requests for the same user
When   both requests arrive within 50ms of each other
Then   both requests return HTTP 200 with valid tokens
  And  only one session record is created in the database
```

**Purpose:** These are where production bugs live. Writing them explicitly forces the question: "What actually happens in this case?"

---

### Type 3 — Error Cases

Invalid inputs, missing authorization, downstream failures. The system must fail gracefully — never catastrophically, never in a way that leaks information.

**Template:**
```
Given  [a condition that makes the action invalid or unauthorized]
When   [the user attempts the action]
Then   the system returns [specific error HTTP status]
  And  returns error code [SPECIFIC_CODE]
  And  does NOT [reveal sensitive information / change state / allow access]
```

**Example (user authentication):**
```
Given  a registered user
When   the user submits the login form with an incorrect password
Then   the system returns HTTP 401 with error code INVALID_CREDENTIALS
  And  does NOT reveal whether the email address exists
  And  increments the failed_attempt_count for the account

Given  a user whose account is locked after 5 failed attempts
When   the user submits the login form with correct credentials
Then   the system returns HTTP 403 with error code ACCOUNT_LOCKED
  And  does NOT return a token
  And  does NOT reset the failed_attempt_count

Given  the token validation service is unavailable
When   the user submits the login form
Then   the system returns HTTP 503 with error code SERVICE_UNAVAILABLE
  And  does NOT expose internal error details in the response body
```

**Purpose:** Security, resilience, and user trust. Error cases are often skipped — which is exactly why security vulnerabilities and bad UX happen.

---

## Writing Rules

| Rule | Why |
|---|---|
| One `When` per scenario | Multiple triggers make the test ambiguous — which one caused the outcome? |
| Specific HTTP status in `Then` | "returns an error" is not testable. "returns HTTP 401" is. |
| Named error codes | "returns error message" is not testable. "returns error code INVALID_CREDENTIALS" is. |
| Explicit `does NOT` | If something must not happen, write it. Never leave it implicit. |
| No implementation in `Then` | "Then the database record is marked INACTIVE" is fine. "Then `UPDATE users SET status='INACTIVE'` runs" is not. |
| One scenario per behavior | Don't combine happy path + edge case in one scenario. |

---

## From Scenario to Test

Every `Given/When/Then` scenario translates directly to a test. The translation is mechanical:

```
Given  a registered user with valid credentials
→ test setup: create user in DB, prepare valid credentials

When   the user submits the login form
→ action: POST /auth/login { email, password }

Then   the system returns HTTP 200 with a JWT access token
→ assertion: expect(response.status).toBe(200)
→ assertion: expect(response.body.token).toBeDefined()

And  the token expires in 1 hour
→ assertion: const decoded = jwt.decode(token); expect(decoded.exp - decoded.iat).toBe(3600)
```

If you can't translate a clause directly into an assertion, the clause is still vague. Rewrite it.

---

## Scenario Coverage Checklist

Before finalizing a spec, verify:

- [ ] Every functional requirement has at least one happy path scenario
- [ ] Every input boundary has an edge case scenario
- [ ] Every error condition has an error case scenario
- [ ] Every `Then` clause can be turned into a concrete assertion
- [ ] Every `does NOT` clause has a corresponding negative assertion
