# root-cause.md

> **Step 4 of the Runtime Gateflow.**
> Evidence from the log is mapped to specific files and lines.
> No fix is proposed here — only the cause, with proof.

---

## Root Cause #1 — No pagination on API call

**Evidence:** `evidence-log.md` #1 — `fetchProducts` returns 842KB, takes 1,623ms  
**Spec clause violated:** `SHALL render within 1,500ms`

**Location:** `src/api/products.js:42`

```js
// CURRENT — no pagination params
const fetchProducts = async () => {
  const response = await fetch('/api/v1/products');  // ← returns ALL products
  return response.json();
};
```

**Why this causes the violation:**  
The API endpoint supports pagination (`?page=0&size=20`) but the client never sends these params. Every page load fetches the entire product catalog — 200 items, 842KB — regardless of what the user can see. The API response is 40x larger than it needs to be for the initial render.

**Contributing factor:** The API also has no `Cache-Control` header, so the full payload is re-fetched on every navigation.

---

## Root Cause #2 — Unvirtualized list renders all 200 cards synchronously

**Evidence:** `evidence-log.md` #1 — `renderProductCard` called 200 times, 687ms total  
**Spec clause violated:** `SHALL render within 1,500ms`

**Location:** `src/components/ProductList.jsx:18`

```jsx
// CURRENT — renders every product in a single synchronous pass
return (
  <div className="product-grid">
    {products.map(p => <ProductCard key={p.id} product={p} />)}  {/* ← 200 DOM nodes */}
  </div>
);
```

**Why this causes the violation:**  
200 `ProductCard` components are mounted synchronously on every render. Each card renders an image, price, badge, and button — approximately 12 DOM nodes each. That's 2,400 DOM nodes created in a single render pass, blocking the main thread for 687ms.

**Note:** This is a compounding factor. Even with pagination fixed (root cause #1), a page of 20 items would render in ~68ms. The real fix needs both changes.

---

## Root Cause #3 — Unhandled error in empty-state branch

**Evidence:** `evidence-log.md` #3 — `Cannot read properties of undefined (reading 'length')` at `ProductList.jsx:34`  
**Spec clause violated:** `SHALL display "No products found."` + `SHALL NOT display spinner indefinitely`

**Location:** `src/components/ProductList.jsx:34`

```jsx
// CURRENT — crashes when API returns { data: [], message: "..." }
const ProductList = ({ products }) => {
  if (isLoading) return <Spinner />;

  // BUG: products is the full ApiResponse object { data: [], message: "" }
  // but the component expects a raw array. products.length throws when products = object.
  if (products.length === 0) {           // ← TypeError: Cannot read 'length' of undefined
    return <EmptyState message="No products found." />;
  }

  return <div>{products.map(...)}</div>;
};
```

**Why this causes the violation:**  
The API response shape changed from `Product[]` to `ApiResponse<Product[]>` (wrapping the data in `{ data: [], message: "" }`), but `ProductList.jsx` was never updated to unwrap it. When the API returns an empty catalog, `products` is `{ data: [], message: "" }` — an object, not an array. `.length` throws a TypeError, the component crashes mid-render, the `isLoading` state is never cleared, and the spinner stays visible indefinitely.

---

## Root Cause #4 — Redundant API fetches (contributing factor)

**Evidence:** `evidence-log.md` #4 — `fetchProducts` called 3 times on mount  
**Spec clause violated:** `SHALL render within 1,500ms` (contributing, not primary)

**Location:** `src/hooks/useProducts.js:12`

```js
// CURRENT — missing stable dependencies, causes re-fetch on every render
useEffect(() => {
  fetchProducts();
}, [fetchProducts]);  // ← fetchProducts is recreated on every render (not memoized)
                      //   causes infinite re-fetch loop caught only by React's batching
```

**Why this causes the violation:**  
`fetchProducts` is not wrapped in `useCallback`, so a new function reference is created on every render. This triggers `useEffect` to re-run, which calls `fetchProducts` again, which updates state, which triggers a re-render. React's batching prevents an infinite loop in practice, but it still fires 2–3 times on mount instead of once.

---

## Impact Summary

| Root Cause | Violation | File | Line |
|---|---|---|---|
| No pagination params | +1,623ms API time | src/api/products.js | 42 |
| Unvirtualized list | +687ms render time | src/components/ProductList.jsx | 18 |
| API response shape mismatch | Empty state crashes | src/components/ProductList.jsx | 34 |
| Missing useCallback | ×2–3 redundant fetches | src/hooks/useProducts.js | 12 |

**Total time wasted:** 2,310ms on top of the actual render work.  
**Spec says:** 1,500ms.  
**Actual:** 2,847ms.

Fix proposals: see [`fix-proposal.md`](fix-proposal.md)
