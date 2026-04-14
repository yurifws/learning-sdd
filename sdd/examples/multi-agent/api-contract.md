# api-contract.md — API Contract

> **Produced by:** API Agent
> **Consumed by:** UI Agent and QA Agent
> **Status:** LOCKED — UI Agent and QA Agent have started.

---

## 1. Endpoints

### GET /api/tasks

**Auth:** Bearer JWT (any authenticated user)
**Query params:**

| Param | Type | Default | Max | Notes |
|---|---|---|---|---|
| `page` | integer | `0` | — | Zero-indexed |
| `size` | integer | `20` | `100` | Returns 400 if exceeded |

**Success — 200**
```json
{
  "tasks": [
    {
      "id": 1,
      "title": "Build the dashboard",
      "status": "TODO",
      "createdAt": "2025-01-15T10:30:00Z"
    }
  ],
  "page": 0,
  "size": 20,
  "totalCount": 47,
  "totalPages": 3
}
```

**Errors**

| Status | When | Body |
|---|---|---|
| `400` | `size` > 100 | `{ "error": "VALIDATION_ERROR", "message": "size must be at most 100" }` |
| `401` | Missing or invalid JWT | `{ "error": "UNAUTHORIZED" }` |

---

### POST /api/tasks

**Auth:** Bearer JWT (any authenticated user)

**Request body**
```json
{ "title": "string — required, max 255 chars" }
```

**Success — 201**
```json
{
  "id": 42,
  "title": "Build the dashboard",
  "status": "TODO",
  "createdAt": "2025-01-15T10:30:00Z"
}
```

**Errors**

| Status | When | Body |
|---|---|---|
| `400` | title blank or missing | `{ "error": "VALIDATION_ERROR", "message": "title is required" }` |
| `400` | title > 255 chars | `{ "error": "VALIDATION_ERROR", "message": "title must be at most 255 characters" }` |
| `401` | Missing or invalid JWT | `{ "error": "UNAUTHORIZED" }` |

---

## 2. Auth Header

All requests must include:
```
Authorization: Bearer <jwt>
```

The API extracts `userId` from the JWT `sub` claim (integer string). If the JWT is missing, expired, or malformed → `401`.

---

## 3. Error Shape

All error responses use this structure — no stack traces, no Express default error page:
```json
{ "error": "ERROR_CODE", "message": "optional human-readable detail" }
```

---

## 4. Gate C Sign-Off

- [x] Endpoints cover all EARS requirements (list with pagination, create with validation)
- [x] Response shapes are fully defined — no ambiguous fields
- [x] All error cases mapped to status codes and error codes
- [x] Auth mechanism documented (JWT sub claim → userId)

**Status: LOCKED. UI Agent and QA Agent may begin.**
