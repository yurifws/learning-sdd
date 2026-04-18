# Measuring SDD — Metrics That Actually Work

> How to turn "it feels faster" into defensible numbers.  
> For the ROI model, see [`WHY_SDD.md`](WHY_SDD.md). For the rollout plan, see [`ROLLOUT.md`](ROLLOUT.md).

---

## Why Old Metrics Break

In a world where AI can generate mountains of code in seconds, measuring productivity by volume — lines of code, closed tickets — actively encourages building fast and fragile. High output becomes the goal, whether or not that output is correct, robust, or what was actually needed.

SDD shifts the focus from code (output) to spec (intent). The metrics have to follow.

What to stop measuring:
- Lines of code generated
- Number of commits
- Tickets closed (without quality filters)

What to start measuring:
- Quality and clarity of specs
- How often implementation deviates from spec
- How cleanly work passes automated checks

---

## Speed Metrics

### Feature Cycle Time

**Definition:** The total elapsed time from spec approval to the feature being in users' hands.

This is the "dock-to-dock" measurement — not just how fast the code is written, but the entire journey. The core promise of SDD is to shrink this timeline by eliminating the back-and-forth that comes from ambiguous requirements.

**What to expect:** Spec time increases. Debug and rework time decreases sharply. If spec time increases but debug time does not decrease, specs are getting longer without getting clearer.

```
Feature Cycle Time = Spec Approval Date → Production Deploy Date
Target trend:        decreasing quarter over quarter
```

**Benchmark:** A DX 2024 report found developers using AI assistance daily saved an average of 4+ hours per week. A controlled study found a ~56% task completion speed improvement in AI-assisted workflows. These are starting benchmarks — measure your own team's actual lift against them.

### Upstream Ratio

**Definition:** The proportion of total feature time spent in the spec/design phase versus the debug/rework phase.

```
Upstream Ratio = Spec + Design Time / Total Feature Cycle Time
```

A rising upstream ratio with a falling cycle time is the ideal signal: more investment up front, less time wasted at the end.

---

## Quality Metrics

### Drift Rate

**Definition:** The percentage of completed features where the production behavior differs from what was agreed in the spec, without a documented spec update.

```
Drift Rate = Features with undocumented behavioral divergence / Total completed features × 100
```

A drift incident is any time the system does something different in production from what the spec says it should do — a mismatched API contract, a missing field, a wrong error code. This is a direct measurement of architectural health. A healthy system has low drift.

See [`PRODUCTION_RISKS.md`](PRODUCTION_RISKS.md) for why spec drift compounds over time and how to prevent it.

### Defect Classification

Instead of counting bugs in a single bucket, categorize them by root cause. SDD is specifically designed to reduce defects in the first three categories:

| Root cause | Description | SDD impact |
|---|---|---|
| Spec ambiguity | Requirement was unclear; AI or developer guessed wrong | Eliminated by clarification gate |
| Implementation drift | Code deviated from spec without a spec update | Eliminated by drift detection |
| Missing coverage | Scenario or edge case was never specified | Eliminated by test-first gate |
| Environmental / infrastructure | Deployment, config, dependency issues | Not directly affected |
| Regression | Previously working behavior broken by a change | Reduced by automated gate enforcement |

Track the percentage of bugs in each category over time. As SDD matures, the first three categories should trend toward zero.

### Gate First-Pass Rate

**Definition:** The percentage of PRs that pass all automated checks on the first attempt.

```
Gate First-Pass Rate = PRs passing all checks on first submit / Total PRs × 100
```

A high first-pass rate means specs are well-defined and developers know exactly what done looks like before they start. A low first-pass rate means the definition of "ready" for specs needs to be tighter.

### Rework Rate

**Definition:** The percentage of completed tasks that required re-opening due to behavioral issues after the task was marked done.

```
Rework Rate = Tasks reopened for behavioral issues / Total completed tasks × 100
```

A rising rework rate signals that specs are being approved without sufficient clarity. The fix is upstream — a stricter clarification gate, not faster debugging.

---

## ROI Calculation

A four-step model that produces a defensible number for budget conversations:

**Step 1 — Baseline:** What did it cost to deliver a feature before SDD? Measure actual cycle time, rework hours, and defect-fixing time for a representative sample of pre-SDD work.

**Step 2 — Delta:** After SDD adoption, measure the same metrics. Calculate the difference.

**Step 3 — Subtract real costs:** Deduct the actual investment — time spent writing specs, team training, tooling setup. Be honest. Understating costs destroys credibility.

**Step 4 — Net ROI:** What remains is the net return.

```
Net ROI = (Baseline Cost − Post-SDD Cost) − SDD Investment
```

Track this monthly. The first one or two months will likely show a negative or flat return — this is the expected investment dip as the team learns the workflow. The value compounds from month three onward as habits stabilize and rework drops.

**Important:** Set this expectation explicitly before you start measuring. An initial dip is not failure. Presenting it transparently, alongside the trajectory, is how you build credibility with leadership.

---

## Measurement Cadence

Not every metric needs to be checked at the same frequency.

| Cadence | Metric | What you're watching |
|---|---|---|
| Per PR | Gate first-pass rate | Immediate signal that spec quality is holding |
| Weekly | Feature cycle time trend | Is the overall pipeline getting faster? |
| Weekly | Drift rate | Are specs staying in sync with production? |
| Weekly | Rework rate | Is the definition of done working? |
| Monthly | Defect classification breakdown | Which root causes are declining? |
| Monthly | Net ROI | Is the investment compounding as expected? |

---

## Minimum Viable Dashboard

If you are just starting, track these six metrics. They are sufficient to tell a clear, data-driven story about the impact of SDD on speed, quality, and value.

| # | Metric | Answers |
|---|---|---|
| 1 | Feature cycle time | Are we covering more ground, or just spinning faster? |
| 2 | Upstream ratio | Is the effort shifting to where it has the highest leverage? |
| 3 | Drift rate | Is the spec staying as the source of truth? |
| 4 | Gate first-pass rate | Are specs clear enough before implementation starts? |
| 5 | Rework rate | Is the definition of done actually working? |
| 6 | Net ROI (monthly) | Is the investment paying back? |

Add more metrics when you have the tooling and the baseline data to make them meaningful. Starting with six and tracking them consistently is better than tracking twenty inconsistently.

---

## The Goal

Good measurement serves three purposes:

- **Observable** — everyone from the engineer to the executive can see what the process is producing
- **Repeatable** — wins can be reproduced because you know what caused them
- **Defensible** — results are backed by data, not anecdotes

Your tools are getting faster. Your metrics need to keep up.
