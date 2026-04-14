# root-cause.md — Task List Animated Expand/Collapse

> **Step 4 of the Runtime Gateflow.**
> Evidence from the log is mapped to specific files and lines.
> No fix is proposed here — only the cause, with proof.

---

## Root Cause #1 — Layout thrashing: reading `offsetHeight` after writing `style.height`

**Evidence:** `evidence-log.md` #2, #4 — 8 forced reflow events at `TaskCard.jsx:67` during each animation
**Spec clause violated:** `SHALL animate at 60fps (frame duration ≤ 16ms)`

**Location:** `src/components/TaskCard.jsx:61`

```js
// CURRENT — reads layout property after writing a layout-affecting style
animate(targetHeight) {
  const startHeight = this.cardRef.current.offsetHeight;  // line 61: read — OK
  const delta = targetHeight - startHeight;
  let progress = 0;

  const step = () => {
    progress += 0.05;
    const current = startHeight + delta * progress;

    this.cardRef.current.style.height = current + 'px';    // line 67: WRITE — sets style
    const actual = this.cardRef.current.offsetHeight;      // line 68: READ — forces reflow ← BUG

    if (progress < 1) requestAnimationFrame(step);
  };

  requestAnimationFrame(step);
}
```

**Why this causes the violation:**

This is **layout thrashing** — a pattern where the browser is forced to synchronously recalculate layout because a layout property is read immediately after a style write that would affect layout.

When `style.height` is set on line 67, the browser marks the layout as "dirty" — it knows geometry has changed, but it defers the recalculation until it's actually needed (usually at paint time). This deferral is what allows batching and smooth animation.

On line 68, `offsetHeight` is read immediately after. To return the correct value, the browser must flush the pending layout recalculation *right now*, synchronously, before returning. This forces the browser to:

1. Stop JavaScript execution
2. Apply all pending style changes
3. Recalculate the geometry of every element in the document
4. Return `offsetHeight`
5. Resume JavaScript

This recalculation is the forced reflow. It takes approximately 8–12ms per frame on a 4x-throttled CPU. When the animation runs at 60fps (one frame every 16ms), a 10ms forced reflow leaves only 6ms for the rest of the frame work — which is not enough. Frames drop.

**The deeper problem:** `actual` (line 68) is never used — it's a dead read. The forced reflow is happening for no reason.

---

## Root Cause #2 — Animation duration exceeds 300ms spec

**Evidence:** `evidence-log.md` #1 — animation takes 412ms, spec says 300ms
**Spec clause violated:** `SHALL complete the expand animation within 300ms`

**Location:** `src/components/TaskCard.jsx:64`

```js
// CURRENT — progress increments by 0.05 per frame
// At 60fps (16ms/frame): 1.0 / 0.05 = 20 frames × 16ms = 320ms (just over limit)
// At 52fps (19ms/frame): 20 frames × 19ms = 380ms (well over limit due to dropped frames)
progress += 0.05;
```

**Why this causes the violation:**

The animation step increments `progress` by a fixed 0.05 per frame. This assumes 60fps. At 60fps, the animation takes exactly 20 frames × 16ms = 320ms — already 20ms over the 300ms spec. Under real conditions (dropped frames from Root Cause #1), each frame averages 19.2ms, making the actual duration 20 × 19.2ms = 384ms.

The correct approach is to drive animation progress from elapsed time (milliseconds), not frame count. This makes the duration deterministic regardless of frame rate.

---

## Impact Summary

| Root Cause | Violation | File | Line |
|---|---|---|---|
| Layout thrashing: dead `offsetHeight` read after `style.height` write | Frame drops (52fps avg) | src/components/TaskCard.jsx | 68 |
| Frame-count-based progress (assumes 60fps) | Duration 412ms vs 300ms spec | src/components/TaskCard.jsx | 64 |

Fix proposals: see [`fix-proposal.md`](fix-proposal.md)
