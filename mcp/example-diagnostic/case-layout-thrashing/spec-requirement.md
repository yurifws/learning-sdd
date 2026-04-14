# spec-requirement.md — Task List Animated Expand/Collapse

> **Step 1 of the Runtime Gateflow.**
> Every diagnostic session starts here — with a single, specific requirement from the spec.
> Never begin without a spec anchor.

---

## Feature

**Task List — Animated Card Expand/Collapse**

---

## Source Requirement (EARS)

```
WHEN a user clicks to expand or collapse a task card,
  the system SHALL animate the height transition at a consistent 60fps frame rate,
  with no visible frame drops or jank during the animation.

WHEN a user expands a task card,
  the system SHALL complete the expand animation within 300ms.
```

---

## What Triggered This Investigation

**User report:** Multiple users reported that expanding task cards "stutters" or "lags" — especially noticeable when the task list contains more than 20 items. The animation appears smooth on a developer machine but visibly janky in production.

**Why this is a spec issue:**
"Stutters" and "lags" are subjective. The spec defines the testable threshold: 60fps, no frame drops. The diagnostic will produce hard frame-timing data and locate the cause in the code. Subjective reports become objective violations or clearances.

---

## Diagnostic Goal

Produce evidence that either:

1. **Confirms the violation** — frame timing data shows frames consistently above 16ms, with root cause localized to a specific file and line
2. **Clears the requirement** — frames meet the 60fps threshold and the reports are device/environment-specific

---

## Session Scope

| Tool | Permitted actions |
|---|---|
| Playwright | navigate, click, screenshot, evaluate |
| DevTools | startTrace, stopTrace, queryTrace, getFrameTimeline, getConsoleLog |
| Figma | not required |

**Environment:** `http://localhost:3000/tasks`
**Test data:** Task list seeded with 25 items (matches reported scenario)
**Note:** Run traces in throttled CPU mode (4x slowdown) to reproduce production conditions on a developer machine.

---

## Definition of Done

The diagnostic session is complete when:

- [ ] Frame timeline captured — frame durations recorded across a full expand/collapse cycle
- [ ] Forced reflow events identified — file, function, and line recorded
- [ ] Root cause mapped to specific animation code
- [ ] Fix proposed in `fix-proposal.md` with proof steps
- [ ] Spec gap assessed — was the 60fps requirement already enforced anywhere?
