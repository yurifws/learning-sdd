# requirements.md — Client API Requirements

> **Project:** tech.example.api.client
> **Status:** Approved
> **Date:** 2025-01-01

---

## 1. Overview

**Goal:** Expose a REST API to query client data in a secure, auditable way.
**Domain:** Client management — summary, addresses, contacts, plan assignment.
**Architecture:** Hexagonal (Ports & Adapters) — see `CONSTITUTION.md`.

---

## 2. Functional Requirements (EARS Notation)

### 2.1 Ubiquitous (Always true)

```
The API SHALL require a valid JWT Bearer token on every endpoint.
Every endpoint SHALL be annotated with @PreAuthorize using a named role constant from Authorities.
The API SHALL document every endpoint with @Operation, @ApiResponse, and @Tag (OpenAPI 3).
Every response SHALL be wrapped in the standard HTTP status code — no raw exceptions exposed.
```

---

### 2.2 Event-Driven — GET /clients/{idClient}/summary

```
WHEN a GET request is sent to /clients/{idClient}/summary,
  IF the caller has ROLE_CLIENT_QUERY_SUMMARY
    AND the client with the given idClient exists,
  THEN the system SHALL return HTTP 200
    AND the body SHALL conform to ClientSummaryResponseModel.

WHEN a GET request is sent to /clients/{idClient}/summary,
  IF the client with the given idClient does NOT exist,
  THEN the system SHALL return HTTP 404
    AND the body SHALL contain a human-readable error message.
```

---

### 2.3 Event-Driven — GET /clients/{idClient}/addresses

```
WHEN a GET request is sent to /clients/{idClient}/addresses,
  IF the caller has ROLE_CLIENT_QUERY_ADDRESS
    AND the client with the given idClient exists
    AND the client has at least one registered address,
  THEN the system SHALL return HTTP 200
    AND the body SHALL be a non-empty List<AddressResponseModel>.

WHEN a GET request is sent to /clients/{idClient}/addresses,
  IF the client exists AND has no registered addresses,
  THEN the system SHALL return HTTP 200
    AND the body SHALL be an empty list [].

WHEN a GET request is sent to /clients/{idClient}/addresses,
  IF the client with the given idClient does NOT exist,
  THEN the system SHALL return HTTP 404.
```

---

### 2.4 Unwanted Behavior (Security Rules)

```
The system SHALL NOT expose destination URLs or internal identifiers in error responses.
The system SHALL NOT log passwords, tokens, CPF, emails, or any PII.
The system SHALL NOT allow a client to query another client's data.
IF a request is made without a valid JWT,
  THEN the system SHALL return HTTP 401.
IF a request is made with a valid JWT but insufficient role,
  THEN the system SHALL return HTTP 403.
```

---

## 3. Non-Functional Requirements

| Category | Requirement |
|---|---|
| Response time | GET endpoints SHALL respond in < 200ms for standard queries |
| Security | Every endpoint requires JWT. Role hierarchy defined in `Authorities` |
| Logging | `log.info` on entry and exit. `log.warn` on not-found. Never log PII. |
| Mapping | All DTO-to-domain and domain-to-response mapping uses MapStruct only |

---

## 4. Acceptance Criteria

### Summary endpoint
- [ ] Returns HTTP 200 with `ClientSummaryResponseModel` for a valid client
- [ ] Returns HTTP 404 when client ID does not exist
- [ ] Returns HTTP 401 when JWT is missing
- [ ] Returns HTTP 403 when caller lacks `ROLE_CLIENT_QUERY_SUMMARY`

### Addresses endpoint
- [ ] Returns HTTP 200 with a list of addresses for a valid client
- [ ] Returns HTTP 200 with `[]` when client exists but has no addresses
- [ ] Returns HTTP 404 when client ID does not exist
- [ ] Returns HTTP 401 when JWT is missing
- [ ] Returns HTTP 403 when caller lacks `ROLE_CLIENT_QUERY_ADDRESS`
- [ ] Response body does NOT contain any field not in `AddressResponseModel`
