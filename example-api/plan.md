# plan.md — Development Plan

## Status Legend
- ✅ Done
- 🔄 In Progress
- ⏳ Pending

---

## Phase 1 — Foundation
- ✅ Define hexagonal package structure
- ✅ Configure Spring Boot project
- ✅ Set up JPA entities and BaseEntity
- ✅ Set up BaseMapper with MapStruct
- ✅ Define Authorities class with role hierarchy
- ✅ Set up OpenAPI base configuration

---

## Phase 2 — Client Endpoints

### Summary
- ✅ GET `/clients/{idClient}/summary` — Find client summary

### Address
- ⏳ GET `/clients/{idClient}/addresses` — Find client addresses

### Contacts
- ⏳ GET `/clients/{idClient}/contacts` — Find client contacts

### Plan
- ⏳ PUT `/clients/{idClient}/plan` — Update client plan

---

## Phase 3 — Tests
- ⏳ Unit tests for use cases
- ⏳ Integration tests for persistence services
- ⏳ Controller tests with MockMvc

---

## Decisions

| Decision | Reason |
|---|---|
| Hexagonal architecture | Decouples business logic from frameworks — easy to test and swap adapters |
| Native queries (`nativeQuery = true`) | Complex joins and Oracle-style naming not easily expressed in JPQL |
| MapStruct over manual mapping | Type-safe, compile-time checked, zero boilerplate |
| Records for response models | Immutable, concise, with built-in `equals`/`hashCode` |
| `@PreAuthorize` on OpenApi interface | Centralizes security declarations away from implementation |
| Oracle-style column naming | Mirrors enterprise patterns common in real-world corporate projects |
