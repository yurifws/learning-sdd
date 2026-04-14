# root-cause.md — Dashboard LCP Regression

> **Step 4 of the Runtime Gateflow.**
> Evidence from the log is mapped to specific files and lines.
> No fix is proposed here — only the cause, with proof.

---

## Root Cause #1 — Synchronous script blocks the HTML parser

**Evidence:** `evidence-log.md` #3, #4 — `analytics.bundle.js` executes for 3,210ms before the hero image is discovered
**Spec clause violated:** `WHEN /dashboard loads → LCP SHALL be < 2.5s`

**Location:** `public/index.html:8`

```html
<!-- CURRENT (release/2026-04-10) — parser-blocking -->
<head>
  <meta charset="UTF-8">
  <title>Dashboard</title>
  <script src="/analytics.bundle.js"></script>   <!-- ← no defer, blocks parser -->
  <link rel="stylesheet" href="/styles.css">
</head>
```

**Why this causes the violation:**

When the browser encounters a `<script>` tag without `defer` or `async`, it stops parsing HTML, downloads the script, executes it, and only then resumes parsing. `analytics.bundle.js` is 287KB and takes 3,210ms to execute.

During those 3,210ms, the browser has not yet parsed the `<img class="dashboard-hero">` tag — so it doesn't know the hero image exists, and does not start fetching it. The image fetch is delayed by 3,579ms relative to the main branch baseline.

**The regression:** The `defer` attribute was present on main but removed in the `2026-04-10` release. Git diff confirms it was an accidental deletion during a merge conflict resolution.

---

## Root Cause #2 — Hero image preload removed

**Evidence:** `evidence-log.md` #5 — `<link rel="preload">` missing from release branch
**Spec clause violated:** `WHEN /dashboard loads → LCP SHALL be < 2.5s` (contributing)

**Location:** `public/index.html:7`

```html
<!-- CURRENT (release/2026-04-10) — preload missing -->
<head>
  <meta charset="UTF-8">
  <title>Dashboard</title>
  <script src="/analytics.bundle.js"></script>
  <!-- hero image preload removed — browser must discover image via parser -->
</head>

<!-- BASELINE (main) -->
<head>
  <meta charset="UTF-8">
  <title>Dashboard</title>
  <link rel="preload" href="/images/dashboard-hero.webp" as="image">   <!-- early fetch hint -->
  <script src="/analytics.bundle.js" defer></script>
</head>
```

**Why this causes the violation:**

A `<link rel="preload">` tells the browser to fetch the resource as soon as the tag is parsed — before the image element itself is parsed. On main, the hero image fetch starts at 312ms because the preload hint is in `<head>`.

Without the preload hint and with the blocking script in place, the browser can only discover the hero image after the script finishes executing and parsing resumes. This compounds Root Cause #1.

**Note:** Root Cause #2 is a contributing factor. Even with the preload removed, fixing Root Cause #1 alone would bring LCP to approximately 2.1s (hero image fetch starts after parser recovers, ~1.7s earlier). Both fixes together restore the 1.8s baseline.

---

## Impact Summary

| Root Cause | Violation | File | Line | Introduced |
|---|---|---|---|---|
| `defer` removed from analytics script | Parser blocked 3,210ms | public/index.html | 8 | 2026-04-10 |
| `<link rel="preload">` removed | Hero image discovery delayed | public/index.html | 7 | 2026-04-10 |

**LCP on main:** 1,821ms (passing)
**LCP on release/2026-04-10:** 4,103ms (failing — 1,603ms over the 2.5s threshold)

Fix proposals: see [`fix-proposal.md`](fix-proposal.md)
