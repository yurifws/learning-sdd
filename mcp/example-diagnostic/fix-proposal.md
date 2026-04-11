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

**Proof steps:**
1. DevTools `getNetworkLog()` — confirm `/api/v1/products` response size drops below 50KB
2. DevTools `queryTrace` — confirm `fetchProducts` duration drops below 200ms
3. Playwright `evaluate("document.querySelectorAll('.product-card').length")` — confirm 20 cards rendered (first page)

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

**Proof steps:**
1. Playwright `navigate("/products?status=empty")`
2. Playwright `evaluate("document.querySelector('.empty-state')?.textContent")` → `"No products found."`
3. Playwright `evaluate("document.querySelector('.spinner')")` → `null` (spinner gone)
4. DevTools `getConsoleLog()` → zero TypeErrors

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

**Proof steps:**
1. DevTools `getNetworkLog()` — confirm `/api/v1/products` appears exactly once in the request log
2. DevTools `queryTrace` — confirm `fetchProducts` appears once in the call tree

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

## Verification Checklist

After all fixes are applied:

- [ ] DevTools trace: total render time < 1,500ms with 20 products (paginated)
- [ ] DevTools network: `/api/v1/products` called exactly once, response < 50KB
- [ ] Playwright: empty state shows "No products found." within 500ms
- [ ] Playwright: spinner not visible after empty state loads
- [ ] DevTools console: zero TypeErrors on any product list scenario
- [ ] All existing tests pass: `npm test`
- [ ] Spec updated with two new EARS clauses (Gap #1, Gap #2)
