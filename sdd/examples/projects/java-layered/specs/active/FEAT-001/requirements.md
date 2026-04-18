# Requirements — Product API (CRUD)

> **Feature ID:** FEAT-001
> **Date:** YYYY-MM-DD
> **Author:** [Your Name]
> **Status:** Draft | Under Review | Approved

---

## 1. Overview

**Goal:** Expose a REST API to create, read, update, and delete products in the catalog.
**Stakeholder:** Product Team
**Priority:** High

---

## 2. Functional Requirements (EARS Notation)

### 2.1 Ubiquitous (Always true)

```
The API SHALL follow the contract defined in CONSTITUTION.md.
The API SHALL require a valid JWT Bearer token on all endpoints.
The API SHALL return all responses wrapped in ApiResponse<T>.
The API SHALL validate all request bodies using Bean Validation.
```

---

### 2.2 Event-Driven — Create Product

```
WHEN a POST request is sent to /api/v1/products with a valid body,
  the system SHALL persist the product with status ACTIVE
  and SHALL return HTTP 201 with the created product.

WHEN a POST request is sent to /api/v1/products,
  IF a product with the same SKU already exists,
  THEN the system SHALL return HTTP 409 with error code CONFLICT
  AND SHALL NOT persist the duplicate product.
```

---

### 2.3 Event-Driven — Read Product

```
WHEN a GET request is sent to /api/v1/products/{id},
  IF the product exists, the system SHALL return HTTP 200 with the product data.
  IF the product does not exist, the system SHALL return HTTP 404 with error NOT_FOUND.

WHEN a GET request is sent to /api/v1/products,
  the system SHALL return a paginated list of ACTIVE products.
  The system SHALL support query params: page (default 0), size (default 20), sort (default "name,asc").
```

---

### 2.4 Event-Driven — Update Product

```
WHEN a PUT request is sent to /api/v1/products/{id} with a valid body,
  IF the product exists, the system SHALL update all provided fields
  and SHALL return HTTP 200 with the updated product.

  IF the product does not exist, the system SHALL return HTTP 404.
```

---

### 2.5 Event-Driven — Delete Product

```
WHEN a DELETE request is sent to /api/v1/products/{id},
  IF the product exists, the system SHALL set its status to INACTIVE (soft delete)
  AND SHALL NOT physically remove the record from the database.
  AND SHALL return HTTP 200 with message "Product deactivated successfully".

  IF the product does not exist, the system SHALL return HTTP 404.
```

---

### 2.6 Unwanted Behavior (Validation Rules)

```
IF name is null or blank,
  THEN the system SHALL return HTTP 400 with error VALIDATION_ERROR
  AND message "name is required".

IF price is null or less than 0.01,
  THEN the system SHALL return HTTP 400 with error VALIDATION_ERROR
  AND message "price must be greater than 0".

IF sku is null, blank, or longer than 50 characters,
  THEN the system SHALL return HTTP 400 with error VALIDATION_ERROR
  AND message "sku is required and must be at most 50 characters".

IF stock is null or less than 0,
  THEN the system SHALL return HTTP 400 with error VALIDATION_ERROR
  AND message "stock must be zero or greater".
```

---

## 3. Non-Functional Requirements

| Category | Requirement |
|---|---|
| Performance | GET /products must respond in < 300ms for up to 10,000 records |
| Security | All endpoints require JWT. Role: ROLE_ADMIN to create/update/delete |
| Availability | 99.9% uptime |
| Pagination | Default page size 20, max 100 |

---

## 4. Out of Scope

- Product images upload
- Product categories (separate feature)
- Price history tracking
- Multi-currency support

---

## 5. Open Questions

| # | Question | Owner | Due |
|---|---|---|---|
| 1 | Should deleted products appear in order history? | @[owner] | YYYY-MM-DD |

---

## 6. Acceptance Criteria

- [ ] POST /api/v1/products creates a product and returns 201
- [ ] POST with duplicate SKU returns 409
- [ ] GET /api/v1/products returns paginated list
- [ ] GET /api/v1/products/{id} returns 404 for unknown ID
- [ ] PUT updates product and returns updated data
- [ ] DELETE sets status INACTIVE, does not remove from DB
- [ ] All validation rules from 2.6 return 400 with correct messages
- [ ] All endpoints return 401 when JWT is missing
- [ ] Unit tests cover all service methods
- [ ] Integration tests cover all controller endpoints
