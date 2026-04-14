# fix-proposal.md — Dashboard LCP Regression

> **Step 5 of the Runtime Gateflow.**
> Each fix is targeted, references its root cause, and includes proof steps —
> the exact observations that will confirm the violation is resolved.

---

## Fix #1 — Restore `defer` on the analytics script

**Addresses:** Root Cause #1 (`public/index.html:8`)
**Expected impact:** Analytics script no longer blocks the HTML parser. Main thread freed 3,210ms earlier. LCP improves from 4,103ms to approximately 2.1s.

**Change:**
```html
<!-- BEFORE -->
<script src="/analytics.bundle.js"></script>

<!-- AFTER -->
<script src="/analytics.bundle.js" defer></script>
```

**Proof — Given / When / Then:**

```
Given  Fix #1 is applied and the dev server is running on release/2026-04-10
When   the user navigates to /dashboard (Playwright navigate + DevTools startTrace)
Then   DevTools queryTrace shows no long task blocking the main thread before 500ms
  And  DevTools getLCPTimeline shows hero image fetch starts before 500ms
  And  DevTools getLCPTimeline shows LCP < 2,200ms
```

---

## Fix #2 — Restore `<link rel="preload">` for the hero image

**Addresses:** Root Cause #2 (`public/index.html:7`)
**Expected impact:** Hero image fetch starts within the first 350ms of navigation, restoring the 1.8s baseline LCP.

**Change:**
```html
<!-- BEFORE -->
<head>
  <meta charset="UTF-8">
  <title>Dashboard</title>
  <script src="/analytics.bundle.js" defer></script>   <!-- after Fix #1 -->

<!-- AFTER -->
<head>
  <meta charset="UTF-8">
  <title>Dashboard</title>
  <link rel="preload" href="/images/dashboard-hero.webp" as="image">
  <script src="/analytics.bundle.js" defer></script>
```

**Proof — Given / When / Then:**

```
Given  Fix #1 and Fix #2 are applied
When   the user navigates to /dashboard (Playwright navigate + DevTools startTrace)
Then   DevTools getNetworkLog shows /images/dashboard-hero.webp requested before 400ms
  And  DevTools getLCPTimeline shows LCP < 2,000ms
  And  Playwright screenshot at 2,000ms shows the hero image rendered (not blank)
```

---

## Final Verification — Given / When / Then

Both fixes applied. Verify the original spec requirement is met.

### Spec requirement: LCP < 2.5s

```
Given  Fix #1 and Fix #2 are applied and the branch is running locally
When   the user navigates to /dashboard (Playwright navigate + DevTools getLCPTimeline)
Then   DevTools getLCPTimeline shows lcp_ms < 2,500
  And  DevTools getLCPTimeline shows lcp_element is "img.dashboard-hero"
  And  DevTools queryTrace shows no blocking script task before 500ms
  And  Playwright screenshot at 2,000ms shows the hero image fully rendered
```

### Regression check

```
Given  Fix #1 and Fix #2 are applied
When   npm test is run (CI suite)
Then   all tests pass with exit code 0
  And  Web Vitals CI check reports LCP < 2.5s
```

---

## Spec Updates Required

The evidence revealed one spec gap.

### Gap #1 — No CI enforcement of the LCP threshold

The spec clause exists (`requirements.md §3.1`), but there is no automated gate that runs Web Vitals in CI and blocks merges when LCP exceeds 2.5s. This regression reached the release branch undetected because the check only runs in production monitoring — too late.

**Proposed spec addition:**

```
WHEN a pull request targets the main branch,
  IF the branch touches public/index.html, src/pages/Dashboard.*, or /images/,
  THEN CI SHALL run a Lighthouse Web Vitals check
  AND SHALL block the merge IF LCP exceeds 2,500ms.
```

**Action:** Add a Lighthouse CI step (`lighthouserc.js`) with `assert.lcp.maxNumericValue: 2500`. This closes the gap that allowed the `defer` deletion to pass CI undetected.

---

## Session Closure

- [x] LCP trace captured — LCP: 4,103ms, element: `img.dashboard-hero`
- [x] Long task identified — `analytics.bundle.js`, 3,210ms, `public/index.html:8`
- [x] Root cause mapped to `2026-04-10` release changes
- [x] Fix #1 and Fix #2 proposed with proof steps
- [x] Spec gap identified — CI gate missing for LCP threshold
