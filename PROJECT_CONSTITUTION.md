# 🏛️ Project Constitution — `[Project Name]`

> **Version:** 1.0 | **Status:** 🔴 Draft | **Date:** YYYY-MM-DD | **Author:** [Name]  
> This document is the non-negotiable rulebook for this project. All AI agents and developers must follow it before writing a single line of code.

---

## 1. Project Overview
> What is this project? What problem does it solve?

[Write here]

---

## 2. Architectural DNA
> How is this system built? AI must deeply understand this before touching anything.

| Concern | Decision |
|---------|----------|
| Architecture pattern | [e.g., Layered MVC, Hexagonal, Microservices] |
| State management | [e.g., Redux, Spring Beans, stateless] |
| Service boundaries | [e.g., auth, users, orders are separate services] |
| Naming conventions | [e.g., camelCase methods, PascalCase classes, snake_case DB columns] |
| Folder structure | [e.g., controller / service / repository / dto] |
| Approved libraries | [e.g., Spring Boot, Lombok, MapStruct — NO others without approval] |

---

## 3. The Gates (Non-Negotiable Rules)

### 🔵 Simplicity Gate
> If something already exists that can be extended, it MUST be extended.  
> Do NOT create new abstractions, classes, or modules unless strictly required.

- [ ] Reuse before creating
- [ ] Extend before replacing
- [ ] Ask before adding a new dependency

### 🔴 Anti-Abstraction Gate
> Do NOT over-engineer. Build only what the current requirement demands.

- [ ] No speculative generalization ("we might need this later")
- [ ] No unnecessary design patterns
- [ ] No unused interfaces or abstract layers

### 🔒 Security Gate
> These rules can NEVER be broken, under any circumstance.

- [ ] No plain-text passwords or tokens — ever
- [ ] No stack traces in API responses
- [ ] No hardcoded credentials in code or config files
- [ ] Input validation on ALL entry points
- [ ] [Add your own non-negotiable rule]

---

## 4. Technical Blueprint

### 4.1 Data Model
> High-level entities and their relationships.

```
[EntityA] (1) ──── (N) [EntityB]
[EntityB] (N) ──── (1) [EntityC]
```

| Entity | Key Fields | Notes |
|--------|-----------|-------|
| [Name] | id, createdAt, ... | |
| [Name] | | |

### 4.2 API Contracts
> Define endpoints before coding. Changing these requires human approval.

| Method | Endpoint | Request | Response | Auth |
|--------|----------|---------|----------|------|
| POST | `/resource` | `{field: type}` | `{id, ...}` | ✅ |
| GET | `/resource/{id}` | — | `{id, ...}` | ✅ |

### 4.3 Security Plan
> How is auth, authorization and data protection handled for this feature?

[Write here]

### 4.4 Rollout Strategy
> How will this be deployed safely?

- [ ] Feature flag? Yes / No
- [ ] DB migration plan: [describe]
- [ ] Rollback plan: [describe]

---

## 5. Risk Map

> For every major decision, answer these three questions.

| Decision | Worst Case | How We'll Know | Escape Plan |
|----------|------------|----------------|-------------|
| [e.g., Adding library X] | [e.g., Incompatible runtime] | [e.g., Build fails] | [e.g., Revert dependency, use alternative] |
| | | | |

### Library Validation Checklist
Before any new library is added:
- [ ] Compatible with current runtime version?
- [ ] Actively maintained (last commit < 6 months)?
- [ ] License compatible with project?
- [ ] No known critical CVEs?

---

## 6. Do NOT List

- [ ] Do NOT modify `[ClassName / module]` — it is shared
- [ ] Do NOT change existing API contracts without approval
- [ ] Do NOT introduce new frameworks without approval
- [ ] Do NOT refactor code outside the current task scope
- [ ] Do NOT skip the human approval gate

---

## 7. Human Approval Gate ✅

> The plan is frozen and AI execution begins ONLY when every box below is checked.

- [ ] Plan meets all requirements
- [ ] Plan obeys the constitution
- [ ] API contracts are explicit and complete
- [ ] Security is planned, not an afterthought
- [ ] Risk map is filled for every major decision
- [ ] Rollout / rollback strategy is defined
- [ ] Reviewed and signed off by: **[Name]** on **YYYY-MM-DD**

> 🔒 **Intent is now frozen.** The AI agent may begin execution.

---

## 8. Clarification Gate

> 🔴 Resolve ALL before the human approval gate.

- [ ] `[NEEDS CLARIFICATION]` ...
- [ ] `[NEEDS CLARIFICATION]` ...

---

## 9. Out of Scope

- [ ] ...
- [ ] ...
