# verification-gates — Example Walkthrough

> A verification gate is a hard stop. The system cannot move forward until specific,
> predefined conditions are met. This turns quality from a hope into a guarantee.

This example walks through all four gates applied to a single feature:
**`POST /orders` — Place a new order**.

---

## The Agent's Dilemma

AI agents are fast. But speed without structure means:

| Strength | Hidden cost |
|---|---|
| Generates code in seconds | Code may be based on a misunderstood requirement |
| Works autonomously | Context drifts — the original goal gets forgotten |
| Fills in gaps automatically | Silent assumptions get baked into the implementation |
| Scales across tasks | Quality degrades as task size grows |

The answer is not to slow the agent down. It is to build a smarter workflow around it.

---

## The Role Flip

| Role | Responsibility |
|---|---|
| **Human** | Architect — sets the vision, defines constraints, validates the blueprint |
| **AI** | Executor — works at full speed, but only within pre-approved boundaries |

---

## The Four Gates

```
Spec Gate    →  locks down the what and the why — zero ambiguity before planning
      ↓
Plan Gate    →  validates the how — architecture reviewed before any code is written
      ↓
Task Gate    →  each chunk of work is atomic, verifiable, and traceable
      ↓
Runtime Gate →  continuous drift detection — live app vs. original spec
```

Each gate is a hard stop. The next stage does not begin until the gate passes.

---

## Files

| File | What it shows |
|---|---|
| [`spec-gate.md`](spec-gate.md) | Filled spec gate — requirements frozen, ambiguity resolved |
| [`plan-gate.md`](plan-gate.md) | Filled plan gate — architecture approved before implementation |
| [`task-gate.md`](task-gate.md) | Filled task gate — one atomic task with traceability |
| [`runtime-gate.md`](runtime-gate.md) | Filled runtime gate — drift detection log with spec update |
