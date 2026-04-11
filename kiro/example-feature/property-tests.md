# Property-Based Tests — Expired Link Handling

> **Feature ID:** FEAT-042
> **Derived from:** requirements.md (EARS clauses)
>
> Property-based tests are generated directly from EARS requirements.
> Each clause produces a **property** — a rule that must hold for all inputs in a given range,
> not just for the two or three examples you thought to write by hand.
>
> Tool used in this example: **jqwik** (Java property-based testing library for JUnit 5)

---

## How to Read This File

Each section maps one EARS clause to:
1. The **property** it implies
2. The **input domain** (what values should be generated)
3. A **jqwik implementation sketch**

---

## EARS → Property Map

---

### Clause 2.2 — Expired link accessed

**EARS:**
```
WHEN the user sends a GET request to /r/{token},
  IF the link exists AND its expires_at timestamp is in the past,
  THEN the system SHALL return HTTP 410
  AND SHALL NOT redirect the user to the destination URL.
```

**Property:**
> For any valid token that resolves to a link whose `expires_at` is any timestamp
> strictly before the current time, the response is always 410 and never contains
> a `Location` header or the destination URL in the body.

**Input domain:**
- Token: any string matching `[a-zA-Z0-9_-]{1,32}`
- `expires_at`: any `OffsetDateTime` in the range `[epoch, now() - 1ms]`

```java
@Property
@Label("expired link always returns 410 with no destination URL")
void expiredLinkReturns410WithNoDestination(
    @ForAll @StringLength(min = 1, max = 32)
    @AlphaChars @Chars({'-', '_'}) String token,

    @ForAll("pastTimestamps") OffsetDateTime expiresAt
) {
    linkRepository.save(linkWith(token, "https://secret-destination.com", expiresAt));

    ResponseEntity<String> response = client.get("/r/" + token);

    assertThat(response.getStatusCode()).isEqualTo(HttpStatus.GONE);
    assertThat(response.getHeaders().getLocation()).isNull();
    assertThat(response.getBody()).doesNotContain("secret-destination.com");
}

@Provide
Arbitrary<OffsetDateTime> pastTimestamps() {
    return Arbitraries.longs()
        .between(Instant.EPOCH.toEpochMilli(), Instant.now().toEpochMilli() - 1)
        .map(ms -> OffsetDateTime.ofInstant(Instant.ofEpochMilli(ms), ZoneOffset.UTC));
}
```

---

### Clause 2.4 — Active link accessed

**EARS:**
```
WHEN the user sends a GET request to /r/{token},
  IF the link exists AND expires_at is null or in the future,
  THEN the system SHALL issue HTTP 302 to the destination URL
  AND SHALL increment the link's click_count by 1.
```

**Property:**
> For any valid token resolving to a link whose `expires_at` is any timestamp
> strictly after the current time (or null), the response is always 302 to the
> correct destination, and click_count increases by exactly 1.

**Input domain:**
- Token: any valid token string
- `expires_at`: any `OffsetDateTime` in `[now() + 1ms, +∞)` OR null
- Destination URL: any well-formed URL string

```java
@Property
@Label("active link always returns 302 to destination and increments click_count")
void activeLinkReturns302AndIncrementsCount(
    @ForAll @StringLength(min = 1, max = 32)
    @AlphaChars @Chars({'-', '_'}) String token,

    @ForAll("futureOrNullTimestamps") Optional<OffsetDateTime> expiresAt,
    @ForAll("validUrls") String destinationUrl
) {
    ShortLink link = linkRepository.save(
        linkWith(token, destinationUrl, expiresAt.orElse(null))
    );
    long countBefore = link.getClickCount();

    ResponseEntity<Void> response = client.get("/r/" + token);

    assertThat(response.getStatusCode()).isEqualTo(HttpStatus.FOUND);
    assertThat(response.getHeaders().getLocation()).hasToString(destinationUrl);
    assertThat(linkRepository.findByToken(token).get().getClickCount())
        .isEqualTo(countBefore + 1);
}
```

---

### Clause 2.5 — Null expires_at is always active

**EARS:**
```
WHILE a link's expires_at field is null,
  the system SHALL treat the link as permanently active
  and SHALL never return HTTP 410 for it.
```

**Property:**
> For any valid token resolving to a link with `expires_at = null`, the response
> is never 410, regardless of when the request is made.

```java
@Property
@Label("null expires_at is never treated as expired")
void nullExpiryIsNeverExpired(
    @ForAll @StringLength(min = 1, max = 32)
    @AlphaChars @Chars({'-', '_'}) String token
) {
    linkRepository.save(linkWith(token, "https://destination.com", null));

    ResponseEntity<Void> response = client.get("/r/" + token);

    assertThat(response.getStatusCode()).isNotEqualTo(HttpStatus.GONE);
}
```

---

### Clause 2.6 — Invalid token never hits the DB

**EARS:**
```
IF the token contains characters outside [a-zA-Z0-9_-],
  THEN the system SHALL return HTTP 400
  AND SHALL NOT attempt any database lookup.
```

**Property:**
> For any string containing at least one character outside `[a-zA-Z0-9_-]`,
> the response is always 400 and the `click_events` table row count never increases.

**Input domain:**
- Token: any string containing at least one character not in the allowed set

```java
@Property
@Label("invalid token always returns 400 with no DB hit")
void invalidTokenReturns400WithNoDbHit(
    @ForAll("tokensWithInvalidChars") String invalidToken
) {
    long clickCountBefore = clickEventRepository.count();

    ResponseEntity<String> response = client.get("/r/" + invalidToken);

    assertThat(response.getStatusCode()).isEqualTo(HttpStatus.BAD_REQUEST);
    assertThat(clickEventRepository.count()).isEqualTo(clickCountBefore);
}

@Provide
Arbitrary<String> tokensWithInvalidChars() {
    // Generate a valid token, then inject one forbidden character at a random position
    return Arbitraries.strings()
        .withCharRange('!', '/')   // characters just outside the allowed range
        .ofMinLength(1).ofMaxLength(32);
}
```

---

### Clause — Destination URL never in error responses (universal rule)

**EARS:**
```
The system SHALL never expose the destination URL in any error response.
```

**Property:**
> For any 4xx response, the response body never contains the destination URL
> of the corresponding link — regardless of why the error occurred.

```java
@Property
@Label("destination URL is never present in any 4xx response body")
void destinationUrlNeverInErrorBody(
    @ForAll @StringLength(min = 1, max = 32)
    @AlphaChars @Chars({'-', '_'}) String token,

    @ForAll("pastTimestamps") OffsetDateTime expiresAt
) {
    String secret = "https://private-destination-" + token + ".com";
    linkRepository.save(linkWith(token, secret, expiresAt));

    ResponseEntity<String> response = client.get("/r/" + token);

    assertThat(response.getStatusCode().is4xxClientError()).isTrue();
    assertThat(response.getBody()).doesNotContain(secret);
    assertThat(response.getBody()).doesNotContain("private-destination");
}
```

---

## Why Property-Based Tests Matter

A hand-written example test checks one scenario:

```java
// Checks ONE specific date
given: link with expires_at = 2024-01-01T00:00:00Z
when: request made on 2025-01-01
then: 410
```

A property-based test checks the entire domain:

```
For ANY expires_at in [epoch, now() - 1ms]
  the response is ALWAYS 410
  the body NEVER contains the destination URL
```

This is the evidence Kiro is designed to generate.
The EARS clause defines the rule. The property test proves it holds universally.
