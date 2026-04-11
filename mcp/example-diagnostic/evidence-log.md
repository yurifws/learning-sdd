# evidence-log.md

> **Step 3 of the Runtime Gateflow.**
> Every observation is logged here, tagged with the spec clause it relates to.
> Evidence that isn't linked to a requirement is noise.

---

## Session Metadata

| Field | Value |
|---|---|
| Date | 2026-04-11 |
| Environment | http://localhost:3000 |
| Tools declared | Playwright, Chrome DevTools |
| Triggered by | Bug report: "product list takes forever to load" |
| Spec clause | requirements.md §2.2 (performance), §2.3 (empty state) |

---

## Evidence #1 — Performance Trace (200 products)

**Tool:** DevTools `stopTrace` + `queryTrace(duration_gt_ms: 200)`  
**Spec clause:** `WHEN /products loads → SHALL render within 1,500ms`

**Raw result:**
```json
{
  "total_render_ms": 2847,
  "slow_functions": [
    {
      "name": "fetchProducts",
      "duration_ms": 1923,
      "file": "src/api/products.js",
      "line": 42,
      "note": "single call, returns 842KB payload"
    },
    {
      "name": "renderProductCard",
      "duration_ms": 687,
      "called_times": 200,
      "file": "src/components/ProductCard.jsx",
      "line": 18,
      "note": "called synchronously 200 times, no virtualization"
    }
  ]
}
```

**Network log:**
```json
{
  "requests": [
    {
      "url": "/api/v1/products",
      "duration_ms": 1623,
      "status": 200,
      "response_size_kb": 842,
      "note": "no pagination params sent — full catalog returned"
    }
  ]
}
```

**Verdict:** `REQUIREMENT VIOLATED`  
Total render: **2,847ms** — exceeds 1,500ms threshold by 1,347ms.  
Two contributing causes identified (see `root-cause.md`).

---

## Evidence #2 — Screenshot (200 products, 3,000ms after navigation)

**Tool:** Playwright `screenshot()`  
**Spec clause:** `WHEN /products loads → SHALL render within 1,500ms`

```
[screenshot attached: products-load-3000ms.png]

Observation: Page still shows skeleton loader at 3,000ms.
Product grid not visible.
```

**Verdict:** Visual confirmation of the performance violation. Skeleton loader is shown past the 1,500ms threshold.

---

## Evidence #3 — Empty State Test

**Tool:** Playwright `navigate` + `waitForSelector(.product-list, timeout: 3000)` + `evaluate`  
**Spec clause:** `IF catalog is empty → SHALL display "No products found." AND SHALL NOT show spinner indefinitely`

**Playwright session:**
```json
[
  { "tool": "navigate", "args": { "url": "http://localhost:3000/products?status=empty" } },
  { "tool": "waitForSelector", "args": { "selector": ".product-list", "timeout": 3000 } },
  { "tool": "evaluate", "args": { "script": "document.querySelector('.empty-state')?.textContent" } },
  { "tool": "evaluate", "args": { "script": "document.querySelector('.spinner')?.style.display" } },
  { "tool": "screenshot", "args": {} }
]
```

**Results:**
```json
{
  "waitForSelector": "resolved after 2,100ms",
  "empty_state_text": null,
  "spinner_display": "block",
  "screenshot": "[screenshot attached: empty-state-spinner.png]"
}
```

**Console log (DevTools `getConsoleLog`):**
```
[ERROR] Unhandled promise rejection: Cannot read properties of undefined (reading 'length')
  at ProductList.jsx:34
```

**Verdict:** `REQUIREMENT VIOLATED`  
- Empty state message: **not rendered** (`empty-state` element absent)
- Spinner: **still visible** after 3,000ms
- Root cause: unhandled JS error in `ProductList.jsx:34` prevents the empty-state branch from executing

---

## Evidence #4 — Console Errors (both scenarios)

**Tool:** DevTools `getConsoleLog()`  
**Spec clause:** (supporting evidence — not directly mapped to a clause)

```
[ERROR] ProductList.jsx:34 - Cannot read properties of undefined (reading 'length')
[WARN]  No cache-control header on /api/v1/products response
[WARN]  fetchProducts called 3 times on initial render (StrictMode double-invoke + useEffect dependency array issue)
```

**Note:** `fetchProducts` fires 3 times on mount. This triples the API load time in development (StrictMode). In production it fires twice. Either way, the dependency array bug causes redundant fetches.

---

## Summary of Violations

| Clause | Evidence | Verdict |
|---|---|---|
| Render within 1,500ms | Total: 2,847ms (trace #1) | VIOLATED |
| Empty state message rendered | `empty-state` absent (evidence #3) | VIOLATED |
| Spinner not shown indefinitely | Spinner visible after 3,000ms (evidence #3) | VIOLATED |

**Root causes:** see [`root-cause.md`](root-cause.md)
