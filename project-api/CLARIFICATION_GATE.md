# Clarification Gate

> A mandatory checkpoint between receiving a request and starting any planning or coding.
> The agent's job at this stage is to surface assumptions, not resolve them.

---

## The Rule

**Any detail not explicitly stated in the request must be flagged — never guessed.**

Mark every open question with the `[NEEDS_CLARIFICATION]` tag.
No planning. No code. Until every tag is resolved.

---

## Thresholds

| Situation | Action |
|---|---|
| More than 5 open items | Stop — request a deeper alignment session before continuing |
| Any item touches auth, PII, money, permissions, or deletion | Immediate full stop — zero tolerance for ambiguity |

---

## The 5 Question Buckets

Run every request through these five categories. If a bucket has any gap, add a `[NEEDS_CLARIFICATION]` item.

### 1. Users & Roles
- Who can trigger this action?
- Are there role differences in what they see or can do?
- What happens if an unauthorized user tries?

### 2. Data Lifecycle
- Is this a soft delete or hard delete?
- What is the data retention period?
- What happens to related records when a parent is deleted?

### 3. Consistency
- What is the expected behavior under concurrent requests?
- Is idempotency required?
- What should happen on retry?

### 4. Error Contracts
- What HTTP status and error code does each failure return?
- What is the exact error message the client receives?
- Are errors logged, and at what level?

### 5. Security & Privacy
- Is any field considered PII?
- Can this data appear in logs?
- Is there a rate limit or abuse vector to consider?

---

## Spec-Ready Checklist

A specification is only approved for implementation when:

- [ ] All `[NEEDS_CLARIFICATION]` tags are resolved
- [ ] Success criteria are measurable (not "works correctly" — specific inputs and outputs)
- [ ] All user roles and their permissions are explicitly defined
- [ ] All error cases have a defined HTTP status + message
- [ ] The data lifecycle is fully specified (create, update, delete, retention)
- [ ] Security and privacy constraints are stated
- [ ] The Do-Not list is written and reviewed

---

## Do-Not List

Write these before any code is written. Examples:

- Do NOT change the existing user schema
- Do NOT touch the public API contract
- Do NOT physically delete records — soft delete only
- Do NOT expose stack traces in responses
- Do NOT log PII

---

## Core Rule

> When ambiguity is found, stop. Update the spec. Then touch the code.
> The specification is the system of record — always.
