# Design — Product API (CRUD)

> **Feature ID:** FEAT-001
> **Linked:** requirements.md
> **Date:** 2025-01-01

---

## 1. Architecture Overview

```
HTTP Request
    │
    ▼
ProductController          ← handles HTTP, delegates to service
    │
    ▼
ProductService             ← business logic, validation, exceptions
    │
    ▼
ProductRepository          ← JpaRepository, DB access only
    │
    ▼
PostgreSQL (products table)
```

---

## 2. Package Structure

```
com.example.api
├── controller/
│   └── ProductController.java
├── service/
│   ├── ProductService.java          (interface)
│   └── ProductServiceImpl.java
├── repository/
│   └── ProductRepository.java
├── model/
│   └── Product.java                 (JPA entity)
├── dto/
│   ├── ProductRequest.java          (create/update body)
│   ├── ProductResponse.java         (response body)
│   └── ApiResponse.java             (generic envelope)
└── exception/
    ├── ResourceNotFoundException.java
    ├── ConflictException.java
    └── GlobalExceptionHandler.java
```

---

## 3. Data Model

### Entity: `products` table

| Column | Type | Constraints |
|---|---|---|
| id | BIGSERIAL | PK |
| name | VARCHAR(255) | NOT NULL |
| sku | VARCHAR(50) | NOT NULL, UNIQUE |
| description | TEXT | nullable |
| price | NUMERIC(10,2) | NOT NULL, > 0 |
| stock | INTEGER | NOT NULL, >= 0 |
| status | VARCHAR(20) | NOT NULL, DEFAULT 'ACTIVE' |
| created_at | TIMESTAMP | NOT NULL, DEFAULT NOW() |
| updated_at | TIMESTAMP | NOT NULL, auto-set via `@PreUpdate` |

---

## 4. DTOs

### ProductRequest (POST / PUT body)
```json
{
  "name": "string, required, max 255",
  "sku": "string, required, max 50",
  "description": "string, optional",
  "price": "decimal, required, min 0.01",
  "stock": "integer, required, min 0"
}
```

### ProductResponse (all endpoints)
```json
{
  "id": "long",
  "name": "string",
  "sku": "string",
  "description": "string | null",
  "price": "decimal",
  "stock": "integer",
  "status": "ACTIVE | INACTIVE",
  "createdAt": "ISO-8601",
  "updatedAt": "ISO-8601"
}
```

### ApiResponse<T> envelope
```json
{
  "data": "T | null",
  "message": "string"
}
```

### Error response
```json
{
  "error": "ERROR_CODE",
  "message": "Human readable message"
}
```

---

## 5. API Endpoints

| Method | Path | Auth | Description |
|---|---|---|---|
| POST | /api/v1/products | ROLE_ADMIN | Create product |
| GET | /api/v1/products | Any role | List products (paginated) |
| GET | /api/v1/products/{id} | Any role | Get product by ID |
| PUT | /api/v1/products/{id} | ROLE_ADMIN | Update product |
| DELETE | /api/v1/products/{id} | ROLE_ADMIN | Deactivate product |

---

## 6. Sequence Diagrams

> Only Create is shown in full. Read/Update/Delete follow the same pattern:
> Controller delegates to Service → Service calls Repository → maps result → returns ResponseEntity.

### Create Product

```
Client          Controller         Service           Repository
  │                  │                │                   │
  │──POST /products─►│                │                   │
  │                  │──createProduct►│                   │
  │                  │                │──findBySku───────►│
  │                  │                │◄──Optional.empty──│
  │                  │                │──save────────────►│
  │                  │                │◄──Product─────────│
  │                  │◄──ProductResp──│                   │
  │◄──201 Created────│                │                   │
```

---

## 7. Exception Handling

| Exception | HTTP | Error Code |
|---|---|---|
| `ResourceNotFoundException` | 404 | NOT_FOUND |
| `ConflictException` | 409 | CONFLICT |
| `MethodArgumentNotValidException` | 400 | VALIDATION_ERROR |
| `AccessDeniedException` | 403 | FORBIDDEN |
| `Exception` (fallback) | 500 | INTERNAL_ERROR |

---

## 8. Technical Decisions

| Decision | Choice | Reason |
|---|---|---|
| Soft delete | Set status=INACTIVE | Preserve order/audit history |
| Pagination | Spring Pageable | Native, well-tested |
| Validation | Bean Validation + @Valid | Standard, declarative |
| Response envelope | ApiResponse<T> | Consistent contract for frontend |
