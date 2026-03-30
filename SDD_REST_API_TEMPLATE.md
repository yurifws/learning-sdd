# 📄 SDD — `[Project Name]`

> **Version:** 1.0 | **Status:** 🔴 Draft | **Date:** YYYY-MM-DD | **Author:** [Name]

---

## 1. Overview
> What is this API and what problem does it solve?

[Write here]

---

## 2. Goals & Non-Goals

**✅ Goals**
- [ ] ...

**❌ Non-Goals**
- [ ] ...

---

## 3. User Stories

| # | As a... | I want to... | So that... |
|---|---------|--------------|------------|
| US-01 | | | |
| US-02 | | | |

---

## 4. Tech Stack

| Layer | Technology |
|-------|------------|
| Language | |
| Framework | |
| Database | |
| Auth | |
| Hosting | |

---

## 5. API Endpoints

Base URL: `https://api.[domain].com/v1`

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| POST | `/auth/login` | | ❌ |
| GET | `/resource/{id}` | | ✅ |

---

## 6. Data Models

### `[EntityName]`

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `id` | UUID | PK | |
| `createdAt` | TIMESTAMP | NOT NULL | |

---

## 7. Success Criteria (EARS)

> Format: `When [trigger], the system SHALL [behavior].`

- [ ] When ..., the system SHALL ...
- [ ] When ..., the system SHALL ...

---

## 8. Error Handling

**Error response shape:**
```json
{
  "status": 400,
  "error": "Bad Request",
  "message": "...",
  "timestamp": "2025-01-01T12:00:00Z"
}
```

| Status | When |
|--------|------|
| 400 | Validation error |
| 401 | Unauthenticated |
| 403 | Unauthorized |
| 404 | Not found |
| 409 | Conflict |
| 500 | Server error |

---

## 9. Security

| Concern | Approach |
|---------|----------|
| Auth | JWT Bearer |
| Passwords | Bcrypt |
| Rate limiting | |
| HTTPS | TLS 1.2+ |

---

## 10. Do NOT List

- [ ] Do NOT modify `[ClassName]`
- [ ] Do NOT expose stack traces in responses
- [ ] Do NOT store plain-text passwords or tokens
- [ ] Do NOT break backward compatibility on existing endpoints

---

## 11. Clarification Gate

> 🔴 Resolve ALL before starting development.

- [ ] `[NEEDS CLARIFICATION]` ...
- [ ] `[NEEDS CLARIFICATION]` ...

---

## 12. Out of Scope

- [ ] ...
- [ ] ...
