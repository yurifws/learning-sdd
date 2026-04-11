# spec-requirement.md

> **Step 1 of the Runtime Gateflow.**
> Every diagnostic session starts here — with a single, specific requirement from the spec.
> Never begin without a spec anchor.

---

## Feature

**Product List Page — Performance & Empty State**

---

## Source Requirement (EARS)

```
WHEN the user navigates to /products,
  the system SHALL render the full product list within 1,500ms
  as measured from the first byte of the HTML response.

WHEN the user navigates to /products,
  IF the product catalog contains no active products,
  THEN the system SHALL display the message "No products found."
  AND SHALL NOT display a loading spinner indefinitely.
```

---

## What Triggered This Investigation

**Bug report:** Users on the product team reported the product list page "takes forever to load" and occasionally shows a spinner that never resolves when the catalog filter returns zero results.

**Why this is a spec issue, not just a bug:** The report is vague. "Takes forever" is not testable. The Runtime Gateflow will produce hard evidence and, if necessary, sharpen the EARS clauses into something that could have caught this automatically.

---

## Diagnostic Goal

Produce evidence that either:

1. **Confirms the violation** — the page consistently exceeds 1,500ms or the empty state fails to render — with root cause localized to a specific file and line
2. **Clears the requirement** — the page meets the spec and the reports are environmental

Either outcome is useful. Speculation is not.

---

## Session Scope

| Tool | Permitted actions |
|---|---|
| Playwright | navigate, screenshot, evaluate, waitForSelector, getNetworkRequests |
| DevTools | startTrace, stopTrace, queryTrace, getNetworkLog, getConsoleLog |
| Figma | not required for this diagnostic |

**Environment:** `http://localhost:3000`  
**Test data:** Seed script `scripts/seed-products.sql` — 200 active products (performance test), 0 active products (empty state test)

---

## Definition of Done

The diagnostic session is complete when:

- [ ] Performance: trace captured and slow functions identified with file + line references
- [ ] Empty state: screenshot evidence of the failing state captured
- [ ] Root cause mapped to `evidence-log.md` with spec clause citation
- [ ] Fix proposed in `fix-proposal.md` with proof steps
- [ ] If spec was vague: updated EARS clause drafted in `fix-proposal.md`
