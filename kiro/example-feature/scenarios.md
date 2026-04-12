# scenarios.md — Expired Link Feature

> **Feature:** Expired Link Handling
> **This file:** All Given/When/Then scenarios derived from `requirements.md`.
> Written before implementation. One scenario per behavior.

---

## Happy Path Scenarios

### H1: Active link redirects to destination

```
Given  a short link with token "abc123" pointing to "https://destination.com"
  And  the link has no expiry date (expires_at is null)
When   the user sends GET /r/abc123
Then   the system returns HTTP 302
  And  the Location header is "https://destination.com"
  And  the link's click_count increases by 1
  And  a click_event is recorded with status ACTIVE
```

### H2: Active link with future expiry redirects

```
Given  a short link with token "xyz789" pointing to "https://destination.com"
  And  the link's expires_at is 30 days in the future
When   the user sends GET /r/xyz789
Then   the system returns HTTP 302
  And  the Location header is "https://destination.com"
  And  a click_event is recorded with status ACTIVE
```

---

## Edge Case Scenarios

### E1: Link expires exactly at request time boundary

```
Given  a short link whose expires_at is set to exactly now (UTC)
When   the user sends GET /r/{token} at that same instant
Then   the system returns HTTP 410
  And  does NOT redirect the user
```

### E2: Null expires_at is never treated as expired

```
Given  a short link with expires_at set to null
  And  the current date is any date in the future
When   the user sends GET /r/{token}
Then   the system returns HTTP 302
  And  does NOT return HTTP 410 regardless of the current date
```

### E3: click_event recorded even for expired link

```
Given  an expired short link with token "old123"
When   the user sends GET /r/old123
Then   the system returns HTTP 410
  And  a click_event is recorded with status EXPIRED
  And  the link's click_count is NOT incremented
```

### E4: click_event recorded for not-found link

```
Given  no link exists with token "ghost99"
When   the user sends GET /r/ghost99
Then   the system returns HTTP 404
  And  a click_event is recorded with status NOT_FOUND
  And  no click_count is updated anywhere
```

---

## Error Case Scenarios

### ER1: Expired link — 410, destination URL never exposed

```
Given  an expired short link pointing to "https://private-destination.com"
When   the user sends GET /r/{token}
Then   the system returns HTTP 410
  And  the response body contains error code "LINK_EXPIRED"
  And  the response body does NOT contain "private-destination.com"
  And  the response headers do NOT contain a Location header
  And  the response headers do NOT contain the destination URL anywhere
```

### ER2: Invalid token characters — 400, no database lookup

```
Given  a token "bad token!" containing a space and exclamation mark
When   the user sends GET /r/bad token!
Then   the system returns HTTP 400
  And  the response body contains error code "INVALID_TOKEN"
  And  no click_event row is added to the database
  And  no database query is made for this token
```

### ER3: Unknown token — 404, no destination leaked

```
Given  no link exists with token "unknown1"
When   the user sends GET /r/unknown1
Then   the system returns HTTP 404
  And  the response body contains error code "LINK_NOT_FOUND"
  And  the response body does NOT contain any destination URL
```

---

## Scenario Coverage Summary

| Requirement | Happy | Edge | Error |
|---|---|---|---|
| Active link redirects (§2.4) | H1, H2 | E2 | — |
| Expired link → 410 (§2.2) | — | E1, E3 | ER1 |
| Not-found → 404 (§2.3) | — | E4 | ER3 |
| Invalid token → 400, no DB (§2.6) | — | — | ER2 |
| click_event always recorded (§2.1) | H1, H2 | E3, E4 | ER1 |
| Null expires_at → always active (§2.5) | — | E2 | — |

All three scenario types covered for each requirement. ✅
