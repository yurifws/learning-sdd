# EARS Reference

> **EARS** — Easy Approach to Requirements Syntax.
> A simple notation that forces every requirement to be unambiguous, machine-readable, and directly testable.

---

## The Problem EARS Solves

Vague requirements create ambiguity. Ambiguity becomes bugs.

| Vague | EARS |
|---|---|
| "Handle expired links" | `WHEN the user clicks an expired link, the system SHALL return HTTP 410` |
| "The page should be fast" | `WHEN /products loads, the system SHALL respond within 1,500ms at p95` |
| "Validate the input" | `IF name is null or blank, THEN the system SHALL return HTTP 400 with error VALIDATION_ERROR` |

The EARS version has a trigger and a clear outcome. There's nothing to interpret.

---

## The Five EARS Patterns

### 1. Ubiquitous — always true, no conditions

Use for rules that apply in every situation.

```
The system SHALL [outcome].
```

**Example:**
```
The system SHALL require a valid JWT Bearer token on all endpoints.
The system SHALL return all responses wrapped in ApiResponse<T>.
```

---

### 2. Event-Driven — a trigger causes a response

Use for the main happy paths and error paths.

```
WHEN [trigger],
  the system SHALL [outcome].
```

**Example:**
```
WHEN a POST request is sent to /api/v1/products with a valid body,
  the system SHALL persist the product
  AND SHALL return HTTP 201 with the created product.
```

---

### 3. State-Driven — behavior in a specific state

Use when the system behaves differently depending on its current state.

```
WHILE [state],
  the system SHALL [outcome].
```

**Example:**
```
WHILE a user's session is expired,
  the system SHALL redirect all requests to /login.
```

---

### 4. Conditional — a precondition guards the behavior

Use for IF/THEN branching within an event.

```
WHEN [trigger],
  IF [condition],
  THEN the system SHALL [outcome].
```

**Example:**
```
WHEN a POST request is sent to /api/v1/products,
  IF a product with the same SKU already exists,
  THEN the system SHALL return HTTP 409 with error code CONFLICT
  AND SHALL NOT persist the duplicate.
```

---

### 5. Unwanted Behavior — explicitly forbidden paths

Use to make negative requirements explicit. Never leave these implicit.

```
IF [condition],
  THEN the system SHALL NOT [forbidden action].
```

**Example:**
```
IF the link is expired,
  THEN the system SHALL NOT return HTTP 302.
  AND SHALL NOT include the destination URL in any response body or header.
```

---

## Chaining Outcomes

Use `AND SHALL` to chain multiple outcomes from a single trigger. Use `THEN` only once per clause.

```
WHEN [trigger],
  IF [condition],
  THEN the system SHALL [first outcome]
  AND SHALL [second outcome]
  AND SHALL NOT [forbidden action].
```

---

## Writing Rules

| Rule | Why |
|---|---|
| One trigger per clause | Multiple triggers in one clause create ambiguity about which fires when |
| Always name the HTTP status | "return an error" is not testable. "return HTTP 404" is |
| Always name the error code | "return an error message" is not testable. "return error LINK_NOT_FOUND" is |
| Explicit > Implicit | If a behavior is forbidden, write `SHALL NOT`. Don't leave it as "implied" |
| Max length: 30 words per clause | If it needs more, split into two clauses |

---

## EARS in the SDD Workflow

EARS requirements live in `requirements.md` (inside `specs/active/FEAT-XXX/`).

They are written **before any design or code**. The Clarification Gate ([`CLARIFICATION_GATE.md`](CLARIFICATION_GATE.md)) runs first to resolve all ambiguity. Only when every clause is unambiguous does design begin.

Each EARS clause maps directly to:
- An acceptance criterion in `requirements.md` Section 6
- A test case in the implementation

If a clause can't be turned into a test, it's still vague. Rewrite it.

---

## Quick Reference Card

```
Ubiquitous:      The system SHALL ...
Event-driven:    WHEN ..., the system SHALL ...
State-driven:    WHILE ..., the system SHALL ...
Conditional:     WHEN ..., IF ..., THEN the system SHALL ...
Unwanted:        IF ..., THEN the system SHALL NOT ...

Chain outcomes:  AND SHALL ... / AND SHALL NOT ...
```
