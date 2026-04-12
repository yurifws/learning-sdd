# CLARIFICATION_GATE.md — Pre-Spec Ambiguity Resolution

> Run this gate BEFORE writing any spec or starting any task.
> Every open question must be resolved before implementation begins.
> Flag anything unclear with `[NEEDS_CLARIFICATION]` and stop — never guess.

---

## Gate Protocol

For every new endpoint or requirement, go through these 5 buckets.
Any item in a zero-tolerance zone (auth, data ownership, deletion) requires explicit sign-off.

| Bucket | Question type | Examples |
|---|---|---|
| Auth & Roles | Who can call this? What happens without auth? | Which `ROLE_*` constant? 401 vs 403? |
| Data Shape | What does the request/response look like? | Fields, types, nullable? Inner objects? |
| Error Handling | What errors are possible and how are they surfaced? | 404 message, 403 body, validation rules |
| Edge Cases | What are the boundary states? | Empty list vs null? Client with no plan? |
| Data Ownership | Can actor A query data belonging to actor B? | Client querying another client's addresses |

---

## Session — GET /clients/{idClient}/addresses

> Held before writing the spec or task for this endpoint.
> All items below were open questions resolved in this session.

---

### Q1: What fields should `AddressResponseModel` contain?

**Question:** The address table schema is not in the spec. What fields does the API consumer need?

**Options considered:**
- Minimal: `street`, `city`, `state`, `zipCode`
- Full: add `number`, `complement`, `neighborhood`, `country`

**Resolution:** Return the full address. Consumers use this for display AND for shipping integrations.

```
AddressResponseModel fields (all required unless noted):
  - Long id
  - String street
  - String number
  - String complement  ← nullable (unit/apt)
  - String neighborhood
  - String city
  - String state        ← 2-char code (e.g., SP, RJ)
  - String zipCode      ← raw digits, no formatting in the API
  - String country      ← ISO 3166-1 alpha-2 (e.g., BR, US)
```

**Owner:** @product
**Resolved:** 2025-01-01

---

### Q2: Does the address table have a direct FK to client or a join table?

**Question:** Is `T_ADDRESS.IDCLIENT` a direct FK, or is there an association table (e.g., `T_CLIENT_ADDRESS`)?

**Options considered:**
- Direct FK: `T_ADDRESS.IDCLIENT = T_CLIENT.IDCLIENT`
- Join table: `T_CLIENT_ADDRESS (IDCLIENT, IDADDRESS)`

**Resolution:** Direct FK. The address table has `IDCLIENT` as a direct FK column. A client can have 0–N addresses.

**Impact on implementation:**
- `ClientRepository` native query: `SELECT * FROM T_ADDRESS WHERE IDCLIENT = :idClient`
- No join table entity needed

**Owner:** @dba
**Resolved:** 2025-01-01

---

### Q3: What is the correct behavior when the client exists but has zero addresses?

**Question:** Should `[]` (empty list) and `404` both be valid answers, or is one preferred?

**Resolution:** Return `HTTP 200` with `[]`. The client exists — "no addresses" is a valid data state, not an error. Returning 404 would imply the client was not found, which would be misleading.

**EARS clause added to requirements.md (§2.3).**

**Owner:** @team
**Resolved:** 2025-01-01

---

### Q4: Who is authorized to call this endpoint?

**Question:** Should addresses use the same role as summary (`ROLE_CLIENT_QUERY_SUMMARY`) or a separate role?

**Resolution:** Separate role: `ROLE_CLIENT_QUERY_ADDRESS`. Address data is more sensitive than summary (contains PII — physical location). Access should be grantable independently.

**Impact:** Add `ROLE_CLIENT_QUERY_ADDRESS` constant to `Authorities`.

**Owner:** @security-lead
**Resolved:** 2025-01-01

---

### Q5: Can a client query another client's addresses?

**Zero-tolerance zone — data ownership.**

**Question:** Is there any scenario where a caller with a valid JWT and correct role should be able to query addresses for a client they are not?

**Resolution:** No. The `idClient` in the path is always the subject of the query. The caller's own identity is not relevant — authorization is role-based only in this API. A user with `ROLE_CLIENT_QUERY_ADDRESS` can query any client's addresses (admin use case, not self-serve consumer).

**Owner:** @security-lead
**Resolved:** 2025-01-01

---

## Gate Sign-Off

| Item | Status |
|---|---|
| All 5 buckets reviewed | ✅ |
| AddressResponseModel fields defined | ✅ |
| DB schema confirmed (direct FK) | ✅ |
| Empty-list behavior documented | ✅ |
| Authorization role defined | ✅ |
| Data ownership rule confirmed | ✅ |
| No unresolved `[NEEDS_CLARIFICATION]` items | ✅ |

> Gate passed. Implementation of `GET /clients/{idClient}/addresses` may begin.
