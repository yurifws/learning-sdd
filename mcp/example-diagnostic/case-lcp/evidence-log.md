# evidence-log.md — Dashboard LCP Regression

> **Step 3 of the Runtime Gateflow.**
> Every observation is logged here, tagged with the spec clause it relates to.
> Evidence that isn't linked to a requirement is noise.

---

## Session Metadata

| Field | Value |
|---|---|
| Date | 2026-04-13 |
| Environment | http://localhost:3000 (release/2026-04-10 branch) |
| Tools declared | Playwright, Chrome DevTools |
| Triggered by | Web Vitals alert: LCP 1.8s → 4.1s after 2026-04-10 release |
| Spec clause | requirements.md §3.1 — LCP SHALL be < 2.5s |

---

## Evidence #1 — LCP Timeline (release branch)

**Tool:** DevTools `getLCPTimeline`
**Spec clause:** `WHEN /dashboard loads → LCP SHALL be < 2.5s`

**Raw result:**
```json
{
  "lcp_ms": 4103,
  "lcp_element": "img.dashboard-hero",
  "lcp_url": "/images/dashboard-hero.webp",
  "navigation_start_ms": 0,
  "lcp_initiated_ms": 3891,
  "note": "hero image fetch did not start until 3,891ms — blocked during HTML parse"
}
```

**Verdict:** `REQUIREMENT VIOLATED`
LCP: **4,103ms** — exceeds 2.5s threshold by 1,603ms.
The hero image (LCP element) did not begin loading until 3,891ms into the navigation. Something blocked the parser from discovering it earlier.

---

## Evidence #2 — LCP Timeline (main branch, baseline)

**Tool:** DevTools `getLCPTimeline`
**Spec clause:** `LCP SHALL be < 2.5s` (baseline comparison)

**Raw result:**
```json
{
  "lcp_ms": 1821,
  "lcp_element": "img.dashboard-hero",
  "lcp_initiated_ms": 312,
  "note": "hero image fetch starts at 312ms — parser-discovered in <head>"
}
```

**Verdict:** `REQUIREMENT MET` on main.
Hero image fetch starts at 312ms on main vs. 3,891ms on release branch. The regression is confirmed and branch-specific.

---

## Evidence #3 — Performance Trace (release branch)

**Tool:** DevTools `startTrace` / `stopTrace` / `queryTrace(task_duration_gt_ms: 500)`
**Spec clause:** `LCP SHALL be < 2.5s` (supporting — identifies the blocking cause)

**Raw result:**
```json
{
  "long_tasks": [
    {
      "name": "EvaluateScript",
      "duration_ms": 3210,
      "source": "/analytics.bundle.js",
      "start_ms": 42,
      "note": "synchronous script execution blocking main thread and HTML parser"
    }
  ],
  "total_blocking_time_ms": 3210
}
```

**Verdict:** A 3,210ms synchronous script execution (`analytics.bundle.js`) starts at 42ms and holds the main thread until 3,252ms. During this window, the HTML parser is blocked — it cannot discover the `<img class="dashboard-hero">` tag and therefore cannot initiate the image fetch.

---

## Evidence #4 — Network Log (release branch)

**Tool:** DevTools `getNetworkLog`
**Spec clause:** (supporting — no direct clause, confirms sequence)

**Raw result:**
```json
{
  "requests": [
    {
      "url": "/analytics.bundle.js",
      "start_ms": 42,
      "duration_ms": 290,
      "size_kb": 287,
      "initiator": "parser",
      "note": "synchronous <script> in <head> — parser blocked until execution completes"
    },
    {
      "url": "/images/dashboard-hero.webp",
      "start_ms": 3891,
      "duration_ms": 212,
      "size_kb": 48,
      "note": "hero image fetch delayed by 3,579ms vs. baseline"
    }
  ]
}
```

**Verdict:** On the release branch, `analytics.bundle.js` (287KB) is loaded as a synchronous parser-blocking script. The hero image is not requested until 3,891ms — nearly 3.6 seconds later than on main.

---

## Evidence #5 — Diff: `index.html` between main and release branch

**Tool:** Playwright `evaluate` (reads `document.head.innerHTML`)
**Spec clause:** (supporting — identifies the code change)

**Raw result:**
```html
<!-- main branch -->
<link rel="preload" href="/images/dashboard-hero.webp" as="image">
<script src="/analytics.bundle.js" defer></script>

<!-- release/2026-04-10 branch -->
<script src="/analytics.bundle.js"></script>
<!-- preload link is missing -->
```

**Verdict:** Two regressions introduced in the `2026-04-10` release:
1. The `defer` attribute was removed from the analytics script tag
2. The `<link rel="preload">` for the hero image was removed

Root causes: see [`root-cause.md`](root-cause.md)

---

## Summary of Violations

| Clause | Evidence | Verdict |
|---|---|---|
| LCP < 2.5s | LCP: 4,103ms (evidence #1) | VIOLATED |

**Confirmed baseline:** main branch meets the spec (LCP: 1,821ms). The regression is isolated to `2026-04-10`.

**Root causes:** see [`root-cause.md`](root-cause.md)
