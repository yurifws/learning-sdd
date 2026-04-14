# fix-proposal.md — Task List Animated Expand/Collapse

> **Step 5 of the Runtime Gateflow.**
> Each fix is targeted, references its root cause, and includes proof steps —
> the exact observations that will confirm the violation is resolved.

---

## Fix #1 — Remove the dead `offsetHeight` read (eliminate layout thrashing)

**Addresses:** Root Cause #1 (`src/components/TaskCard.jsx:68`)
**Expected impact:** Forced reflows eliminated. Frame duration drops from ~19ms to ~5ms. Average fps: 52 → 60+.

**Change:**
```js
// BEFORE — forced reflow on every frame
const step = () => {
  progress += 0.05;
  const current = startHeight + delta * progress;

  this.cardRef.current.style.height = current + 'px';    // WRITE
  const actual = this.cardRef.current.offsetHeight;      // READ — dead variable, forces reflow
                                                          // ← DELETE THIS LINE

  if (progress < 1) requestAnimationFrame(step);
};

// AFTER — no reads after writes, no forced reflow
const step = () => {
  progress += 0.05;
  const current = startHeight + delta * progress;

  this.cardRef.current.style.height = current + 'px';    // WRITE only

  if (progress < 1) requestAnimationFrame(step);
};
```

**Proof — Given / When / Then:**

```
Given  Fix #1 is applied and the task list has 25 items
When   the user clicks to expand a task card (Playwright click + DevTools startTrace)
Then   DevTools queryTrace shows zero forced_reflow events
  And  DevTools getFrameTimeline shows average frame duration < 16ms
  And  DevTools getFrameTimeline shows no frame above 20ms
```

---

## Fix #2 — Drive animation progress from elapsed time, not frame count

**Addresses:** Root Cause #2 (`src/components/TaskCard.jsx:64`)
**Expected impact:** Animation duration becomes exactly 300ms regardless of frame rate.

**Change:**
```js
// BEFORE — frame-count-based (duration varies with fps)
animate(targetHeight) {
  const startHeight = this.cardRef.current.offsetHeight;
  const delta = targetHeight - startHeight;
  let progress = 0;

  const step = () => {
    progress += 0.05;                                      // ← assumes 60fps
    const current = startHeight + delta * progress;
    this.cardRef.current.style.height = current + 'px';
    if (progress < 1) requestAnimationFrame(step);
  };

  requestAnimationFrame(step);
}

// AFTER — time-based (duration is always exactly DURATION_MS)
const DURATION_MS = 300;

animate(targetHeight) {
  const startHeight = this.cardRef.current.offsetHeight;  // read before any writes
  const delta = targetHeight - startHeight;
  const startTime = performance.now();

  const step = (timestamp) => {
    const elapsed = timestamp - startTime;
    const progress = Math.min(elapsed / DURATION_MS, 1);   // clamp to [0, 1]
    const current = startHeight + delta * progress;

    this.cardRef.current.style.height = current + 'px';   // write only

    if (progress < 1) requestAnimationFrame(step);
    else this.cardRef.current.style.height = targetHeight + 'px';  // snap to final value
  };

  requestAnimationFrame(step);
}
```

**Proof — Given / When / Then:**

```
Given  Fix #1 and Fix #2 are applied
When   the user clicks to expand a task card (Playwright click + DevTools startTrace)
Then   DevTools getFrameTimeline shows animation_duration_ms < 320
  And  DevTools getFrameTimeline shows animation_duration_ms > 280
  And  DevTools getFrameTimeline shows average frame duration < 16ms
```

---

## Alternative Fix — Replace JS animation with CSS transition (preferred if no custom easing needed)

**Addresses:** Both Root Causes #1 and #2
**Expected impact:** Browser handles timing natively — zero JS execution on the main thread per frame. Eliminates both layout thrashing and duration variance.

**Change:**
```css
/* styles/TaskCard.css */
.task-card {
  overflow: hidden;
  transition: height 300ms ease-in-out;   /* browser handles it — no JS needed */
}
```

```js
// TaskCard.jsx — remove the entire animate() method
// Replace with a simple height toggle:
toggleExpand() {
  const card = this.cardRef.current;
  const targetHeight = this.state.expanded ? 0 : this.state.fullHeight;
  card.style.height = targetHeight + 'px';    // single write — CSS transition takes over
  this.setState({ expanded: !this.state.expanded });
}
```

**Why this is preferred:** CSS transitions run on the browser's compositor thread — they do not block the main thread at all. The browser guarantees 60fps for transitions as long as the animating property is `height`, `opacity`, or `transform`. Layout thrashing is structurally impossible because JavaScript is not involved in the per-frame computation.

**When to use the JS fix instead:** Only if you need a custom easing curve (e.g. spring physics) that CSS `cubic-bezier` cannot express.

**Proof — Given / When / Then:**

```
Given  CSS transition fix is applied (transition: height 300ms ease-in-out)
When   the user clicks to expand a task card (Playwright click + DevTools startTrace)
Then   DevTools queryTrace shows zero JavaScript execution events during the animation
  And  DevTools queryTrace shows zero forced_reflow events
  And  DevTools getFrameTimeline shows animation_duration_ms between 290ms and 310ms
  And  DevTools getFrameTimeline shows all frame durations < 10ms
```

---

## Final Verification — Given / When / Then

All fixes applied (JS fix or CSS alternative). Verify both original spec requirements are met.

### Spec requirement 1: animation at 60fps

```
Given  fixes are applied and the task list has 25 items (4x CPU throttle)
When   the user clicks to expand a task card (Playwright click + DevTools startTrace)
Then   DevTools getFrameTimeline shows average frame duration < 16ms
  And  DevTools getFrameTimeline shows zero frames above 20ms
  And  DevTools queryTrace shows zero forced_reflow events
```

### Spec requirement 2: animation completes within 300ms

```
Given  fixes are applied
When   the user clicks to expand a task card (Playwright click + DevTools getFrameTimeline)
Then   DevTools getFrameTimeline shows animation_duration_ms < 320
  And  Playwright screenshot at 350ms shows card fully expanded (no mid-state)
```

### Regression check

```
Given  fixes are applied
When   npm test is run
Then   all tests pass with exit code 0
```

---

## Spec Updates Required

### Gap #1 — "Layout thrashing" not detectable from the spec alone

The spec requires 60fps but does not specify *how* the animation must be implemented. A reviewer reading only the spec cannot tell that a JS-based animation is more risky than a CSS transition.

**Proposed spec addition (to `CONSTITUTION.md` or `design.md`):**

```
Animations SHALL use CSS transitions or CSS animations whenever possible.
JavaScript-driven animations SHALL NOT read layout properties (offsetHeight,
offsetWidth, getBoundingClientRect) after writing layout-affecting style properties
in the same call stack.

Rationale: synchronous layout reads after style writes force browser reflows
on the main thread, causing frame drops that violate the 60fps requirement.
```

This turns a runtime rule into a design-time constraint — reviewers and the AI can catch it before it ships.

---

## Session Closure

- [x] Frame timeline captured — average 19.2ms/frame, 52fps average
- [x] Forced reflow events identified — 8 reflows at `TaskCard.jsx:68`, dead `offsetHeight` read
- [x] Root cause mapped to specific animation code
- [x] Fix #1 (remove dead read), Fix #2 (time-based progress), and CSS alternative proposed with proof steps
- [x] Spec gap identified — design constraint for JS animation not in CONSTITUTION.md
