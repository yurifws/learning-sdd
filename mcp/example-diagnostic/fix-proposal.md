# fix-proposal.md

> **Step 5 of the Runtime Gateflow.**
> Each fix is targeted, references its root cause, and includes proof steps —
> the exact observations that will confirm the violation is resolved.

---

## Fix #1 — Add pagination to API call

**Addresses:** Root Cause #1 (`src/api/products.js:42`)  
**Expected impact:** Reduces API payload from 842KB to ~42KB. API call time: 1,623ms → ~80ms.

**Change:**
```js
// BEFORE
const fetchProducts = async () => {
  const response = await fetch('/api/v1/products');
  return response.json();
};

// AFTER
const fetchProducts = async (page = 0, size = 20) => {
  const response = await fetch(`/api/v1/products?page=${page}&size=${size}`);
  return response.json();
};
```

**Proof — Given / When / Then:**

```
Given  Fix #1 is applied and the dev server is running
When   the user navigates to /products (Playwright navigate)
Then   DevTools getNetworkLog() shows /api/v1/products response size < 50KB
  And  DevTools queryTrace shows fetchProducts duration < 200ms
  And  Playwright evaluate('.product-card').length returns 20
```

---

## Fix #2 — Unwrap API response shape in ProductList

**Addresses:** Root Cause #3 (`src/components/ProductList.jsx:34`)  
**Expected impact:** Empty state renders correctly. Spinner resolves.

**Change:**
```jsx
// BEFORE
const ProductList = ({ products }) => {
  if (isLoading) return <Spinner />;
  if (products.length === 0) {           // ← crashes on ApiResponse object
    return <EmptyState message="No products found." />;
  }
  return <div>{products.map(...)}</div>;
};

// AFTER
const ProductList = ({ response }) => {
  if (isLoading) return <Spinner />;
  const products = response?.data ?? [];  // unwrap ApiResponse<Product[]>
  if (products.length === 0) {
    return <EmptyState message="No products found." />;
  }
  return <div>{products.map(...)}</div>;
};
```

**Proof — Given / When / Then:**

```
Given  Fix #2 is applied and the catalog contains zero active products
When   the user navigates to /products (Playwright navigate)
Then   Playwright evaluate('.empty-state').textContent returns "No products found."
  And  Playwright evaluate('.spinner') returns null (spinner absent)
  And  DevTools getConsoleLog() contains zero TypeErrors
```

---

## Fix #3 — Memoize fetchProducts to prevent redundant fetches

**Addresses:** Root Cause #4 (`src/hooks/useProducts.js:12`)  
**Expected impact:** API called once on mount instead of 2–3 times.

**Change:**
```js
// BEFORE
const fetchProducts = async () => { ... };  // recreated every render

useEffect(() => {
  fetchProducts();
}, [fetchProducts]);  // unstable dependency

// AFTER
const fetchProducts = useCallback(async (page = 0, size = 20) => {
  const response = await fetch(`/api/v1/products?page=${page}&size=${size}`);
  return response.json();
}, []);  // stable — no dependencies that change

useEffect(() => {
  fetchProducts();
}, [fetchProducts]);  // now stable
```

**Proof — Given / When / Then:**

```
Given  Fix #3 is applied and the page is loaded fresh (no cache)
When   the user navigates to /products (Playwright navigate)
Then   DevTools getNetworkLog() shows /api/v1/products appearing exactly once
  And  DevTools queryTrace shows fetchProducts appearing exactly once in the call tree
```

---

## Fix #4 — Virtualize product list (deferred)

**Addresses:** Root Cause #2 (`src/components/ProductList.jsx:18`)  
**Expected impact:** Render time for 200 items: 687ms → ~30ms.

**Decision:** After fixes #1–3, the initial page load renders only 20 items (paginated). The 687ms render time applies only when all 200 items are shown at once, which won't happen in normal usage after Fix #1.

**Action:** Add a `[NEEDS_CLARIFICATION]` item to the spec for infinite scroll vs. pagination. If infinite scroll is chosen, implement `react-window` virtualization at that time.

```
[NEEDS_CLARIFICATION] Should /products support infinite scroll (loads more as user scrolls)
or classic pagination (numbered pages)? Virtualization is only needed for infinite scroll.
Owner: @product | Due: 2026-04-18
```

---

## Spec Updates Required

The evidence revealed two gaps in the original spec. These must be updated before closing this diagnostic session.

### Gap #1 — "1,500ms" threshold was assumed, not stated

The original spec said "the page should be fast" in the bug report. The EARS clause was written as part of this diagnostic. It is now precise and should be added to the feature spec officially:

```
WHEN the user navigates to /products,
  the system SHALL render the first page of results within 1,500ms
  as measured from the first byte of the HTML response
  under a standard broadband connection (10Mbps+).
```

### Gap #2 — API response shape not in the spec

The `ProductList` component broke because the API response shape changed from `Product[]` to `ApiResponse<Product[]>` and no spec clause defined this contract. Add to the spec:

```
The API SHALL return all list responses wrapped in ApiResponse<T>:
  { "data": T[], "message": string }
Consumers SHALL always unwrap the data field before accessing array properties.
```

---

## Final Verification — Given / When / Then

All three fixes applied. Verify the original spec requirements are now met.

### Spec requirement 1: render within 1,500ms

```
Given  fixes #1, #2, #3 are applied and the catalog has 200 products
When   the user navigates to /products (Playwright navigate + DevTools startTrace)
Then   DevTools stopTrace shows total render time < 1,500ms
  And  DevTools getNetworkLog shows /api/v1/products response size < 50KB
  And  DevTools getNetworkLog shows /api/v1/products called exactly once
  And  Playwright screenshot shows the product grid visible (no spinner)
```

### Spec requirement 2: empty state renders correctly

```
Given  fixes #1, #2, #3 are applied and the catalog has zero active products
When   the user navigates to /products
Then   Playwright evaluate('.empty-state').textContent returns "No products found."
  And  Playwright evaluate('.spinner') returns null
  And  DevTools getConsoleLog returns zero errors
```

### Regression: existing tests still pass

```
Given  fixes #1, #2, #3 are applied
When   npm test is run
Then   all tests pass with exit code 0
```

### Spec updated

- [ ] EARS clause added for 1,500ms render threshold (Gap #1)
- [ ] EARS clause added for ApiResponse<T> wrapper contract (Gap #2)
