# Lessons Learned — Product API (CRUD)

> Check this file BEFORE fixing any error.
> Update this file AFTER fixing any error.

---

## How to Use

**Before debugging:** scan the Error Log and Known Issues below — has this been seen before?

**After fixing:** add a row to the Error Log with task, error, root cause, fix, and date.

---

## Error Log

| # | Task | Error | Root Cause | Fix | Date |
|---|---|---|---|---|---|
| — | — | No entries yet | — | — | — |

---

## Known Issues & Gotchas

> Add entries here as you discover them. These are reusable across features.

- **`LazyInitializationException` on JSON serialization** →
  JPA session closed before relationship was loaded.
  Fix: add `@Transactional` to the service method, or map to DTO before closing the session.

- **`HttpMessageNotWritableException` on `Page<T>`** →
  Spring's `Page` object does not serialize cleanly by default.
  Fix: map `Page<Entity>` to `Page<ResponseDTO>` before returning, or use `PageImpl`.

- **`403 Forbidden` on all requests despite valid JWT** →
  `SecurityFilterChain` order or role prefix mismatch (`ROLE_ADMIN` vs `ADMIN`).
  Fix: check that `hasRole("ADMIN")` maps to authority `ROLE_ADMIN`, or use `hasAuthority("ROLE_ADMIN")`.

- **`BeanCreationException` on startup after adding `@PreAuthorize`** →
  Method Security not enabled.
  Fix: add `@EnableMethodSecurity` to the main config class.

---

## Patterns That Worked Well

> Note approaches that proved clean and reusable — useful when designing the next feature.

| # | Context | Pattern | Why it worked |
|---|---|---|---|
| — | — | No entries yet | — |

---

## Decisions Made During Implementation

> Decisions not in design.md — things discovered while building.

| # | Context | Decision | Reason |
|---|---|---|---|
| — | — | No entries yet | — |
