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
| 1 | TASK-10 | `LazyInitializationException` serializing `Page<Product>` | JPA session closed before Spring serialized the response; `Page` wraps a proxy | Map `Page<Product>` to `Page<ProductResponse>` inside the service before returning | 2025-01-03 |
| 2 | TASK-14 | `403 Forbidden` on all requests despite valid JWT | Role prefix mismatch: token contained `ADMIN`, security config expected `ROLE_ADMIN` | Changed `hasRole("ADMIN")` → `hasAuthority("ROLE_ADMIN")` in `SecurityFilterChain` | 2025-01-04 |
| 3 | TASK-14 | `BeanCreationException` — `@PreAuthorize` had no effect | `@EnableMethodSecurity` was missing from main config | Added `@EnableMethodSecurity` to `SecurityConfig` | 2025-01-04 |

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
| 1 | TASK-10 | `ProductResponse.from(Product)` static factory | Controller stays thin — no mapping logic leaks out; easy to test in isolation |
| 2 | TASK-11 | `@ParameterizedTest` with `@ValueSource` for auth tests | One test covers all 5 endpoints for the 401/403 check — no duplication |
| 3 | TASK-16 | Testcontainers with `@DynamicPropertySource` for PostgreSQL | Identical schema to prod — caught a `Page` serialization issue that H2 would have missed |

---

## Decisions Made During Implementation

> Decisions not in design.md — things discovered while building.

| # | Context | Decision | Reason |
|---|---|---|---|
| 1 | TASK-10 | Return `Page<ProductResponse>` from service, not `Page<Product>` | Avoids LazyInitializationException; keeps JPA details inside the service layer |
| 2 | TASK-14 | Use `hasAuthority` instead of `hasRole` in SecurityFilterChain | JWT library does not add `ROLE_` prefix automatically — `hasRole` would silently fail |
| 3 | TASK-10 | `findAll` only returns ACTIVE products (not INACTIVE/DELETED) | Confirmed with product team: deactivated products should not be visible in catalog |
