# evidence-log.md — Task List Animated Expand/Collapse

> **Step 3 of the Runtime Gateflow.**
> Every observation is logged here, tagged with the spec clause it relates to.
> Evidence that isn't linked to a requirement is noise.

---

## Session Metadata

| Field | Value |
|---|---|
| Date | 2026-04-13 |
| Environment | http://localhost:3000/tasks (CPU throttle: 4x) |
| Tools declared | Playwright, Chrome DevTools |
| Triggered by | User reports: card expand animation is janky with 20+ items |
| Spec clause | requirements.md §4.2 — animation SHALL run at 60fps, no frame drops |

---

## Evidence #1 — Frame Timeline (expand click, 25-item list)

**Tool:** DevTools `startTrace` → Playwright `click('.task-card:nth-child(3) .expand-btn')` → `stopTrace` → `getFrameTimeline`
**Spec clause:** `WHEN card expands → SHALL animate at 60fps (frame duration ≤ 16ms)`

**Raw result:**
```json
{
  "animation_duration_ms": 412,
  "target_duration_ms": 300,
  "frames": [
    { "frame": 1, "duration_ms": 8 },
    { "frame": 2, "duration_ms": 23 },
    { "frame": 3, "duration_ms": 21 },
    { "frame": 4, "duration_ms": 24 },
    { "frame": 5, "duration_ms": 19 },
    { "frame": 6, "duration_ms": 22 },
    { "frame": 7, "duration_ms": 25 },
    { "frame": 8, "duration_ms": 21 },
    { "frame": 9, "duration_ms": 10 }
  ],
  "average_frame_ms": 19.2,
  "max_frame_ms": 25,
  "frames_over_16ms": 7,
  "fps_average": 52
}
```

**Verdict:** `REQUIREMENT VIOLATED`
Average frame duration: **19.2ms** — corresponds to **~52fps**, below the 60fps threshold.
7 of 9 frames exceed the 16ms budget. Animation duration: 412ms — also exceeds the 300ms spec.
The jank is real and reproducible.

---

## Evidence #2 — Forced Reflow Events in Trace

**Tool:** DevTools `queryTrace(event_type: "forced_reflow")`
**Spec clause:** `animation SHALL run at 60fps` (supporting — identifies cause of frame drops)

**Raw result:**
```json
{
  "forced_reflows": [
    {
      "count": 8,
      "triggered_by": "TaskCard.animate",
      "file": "src/components/TaskCard.jsx",
      "function": "animate",
      "line": 67,
      "note": "offsetHeight read after style.height write — triggers layout recalculation"
    }
  ]
}
```

**Verdict:** 8 forced reflow events during the animation. Each one requires the browser to pause JavaScript execution, flush pending style changes, and calculate the layout of the entire document before returning the `offsetHeight` value. This blocks the main thread on every animation frame.

---

## Evidence #3 — Screenshot at 150ms (mid-animation)

**Tool:** Playwright `screenshot()` at 150ms into the animation
**Spec clause:** `SHALL animate the height transition` (visual confirmation)

```
[screenshot attached: task-card-expand-150ms.png]

Observation: Card is partially expanded but appears to freeze mid-animation.
Height jump visible between frames — not a smooth transition.
```

**Verdict:** Visual confirmation of the jank. The animation is not smooth — discrete height jumps are visible, consistent with forced reflow pausing the main thread between frames.

---

## Evidence #4 — Console Log

**Tool:** DevTools `getConsoleLog`
**Spec clause:** (supporting — no direct clause)

```
[WARN] Avoid reading layout properties (e.g. offsetHeight) after modifying layout-affecting style properties.
       This causes synchronous forced reflows.
       Source: src/components/TaskCard.jsx:67
```

**Verdict:** Browser's built-in performance warning confirms the forced reflow is occurring at `TaskCard.jsx:67`. This matches Evidence #2 exactly.

---

## Summary of Violations

| Clause | Evidence | Verdict |
|---|---|---|
| Animation at 60fps (frame ≤ 16ms) | Average frame: 19.2ms, 7/9 frames over budget | VIOLATED |
| Animation completes within 300ms | Actual duration: 412ms | VIOLATED |

**Root causes:** see [`root-cause.md`](root-cause.md)
