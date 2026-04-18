# Rolling Out SDD — From Pilot to Default

> How to take Spec-Driven Development from an idea to the way your organization ships software.  
> For the business case and ROI model, see [`WHY_SDD.md`](WHY_SDD.md).

---

## The Mental Model: Three Layers

Before you can roll out SDD, your team needs a shared mental model. There are three layers, and they must be completed in order — no skipping ahead.

```
Intent Layer     →  The spec. What are we building, and why?
Design Layer     →  The plan. How does this fit into what already exists?
Execution Layer  →  The tasks. What are the smallest verifiable steps to get there?
```

This maps directly to the files you create for every feature:

| Layer | File | Question it answers |
|---|---|---|
| Intent | `requirements.md` | What must the system do? What must it not do? |
| Design | `design.md` | How does this plug into the existing architecture? |
| Execution | `tasks.md` | What is each atomic step, and how do I verify it? |

The rule is simple: you cannot write `design.md` until `requirements.md` is complete and reviewed. You cannot write `tasks.md` until `design.md` is stable. The process enforces itself through ordering.

---

## The Maturity Model

Teams don't arrive at full SDD maturity overnight. There is a spectrum, and every team starts somewhere on it.

| Stage | What it looks like |
|---|---|
| **Vibe coding** | Prompts drive everything. Code is the deliverable. Docs are an afterthought, if they exist at all. |
| **Structured prompting** | Some specs exist, but they are written after the fact to explain what was built. Tests are added at the end. |
| **Spec-first** | Specs are written before code. Tasks are atomic. Tests are confirmed failing before implementation starts. |
| **Spec-as-source** | The spec is the absolute source of truth. Code is a proven artifact of the spec. Drift is automatically detected and blocked. |

The goal is spec-as-source maturity. The practical starting point for most teams is structured prompting. The gap is closed by following the three-phase rollout below.

---

## The 24-Week Rollout Plan

### Phase 1 — Pilot (Weeks 1–4)

**Goal:** Prove the process has value on something low-risk.

This is not the time to tackle your most critical system. Pick one team, one feature, something real but contained. The objective is to validate the workflow and create the artifacts that will let everyone else follow.

**What to do:**
- Pick one low-risk feature on one team
- Run the full SDD workflow end-to-end: requirements → design → tasks → test-first gate → implementation
- Track metrics from day one (see below)
- Document everything that was confusing or needed adjustment

**What to produce by the end of week 4:**
- A completed feature with a full audit trail: requirement → task → test → PR
- Reusable templates for requirements, design, tasks, and the test-first gate — filled-in examples from the actual feature
- A golden path document: a clear, step-by-step guide showing exactly how to run the process on your stack

**What success looks like:**  
The pilot team can explain the process to a skeptic using real data. The templates are clean enough that another team could follow them without help.

---

### Phase 2 — Scale Consistency (Weeks 5–16)

**Goal:** Make the workflow repeatable and standard, not just a craft project for one team.

**What to do:**
- Expand to more teams, including some work on existing systems (not just greenfield)
- Standardize the workflow so it is tool-agnostic — it should not matter whether the team uses Claude, Cursor, Copilot, or nothing
- Make spec review an official policy: no implementation begins without a reviewed, signed-off spec
- Automate quality checks in CI: spec coverage checks, drift detection, link validation

**What to produce:**
- Shared template library usable by any team
- CI pipeline with at minimum: spec-present check before merge, link validation on spec files
- Updated golden path document incorporating lessons from multiple teams

**What success looks like:**  
A new team can onboard to SDD in one day using the templates and golden path document. Spec coverage is trending up. Drift rate is trending down.

---

### Phase 3 — Default (Weeks 17–24)

**Goal:** SDD is no longer the new thing. It is how work gets done.

**What to do:**
- Roll out to the full organization, including high-stakes systems
- Build enforcement into pipelines: pull requests without a linked, approved spec cannot be merged
- Treat templates and guides as an internal product — own them, version them, improve them
- Provide regular, data-backed updates to leadership showing actual outcomes: rework reduction, defect rates, onboarding time

**What to produce:**
- Hard enforcement in CI/CD: no merge without spec reference
- Quarterly review report: spec coverage, drift rate, DORA metrics trend, rework cost avoided
- SDD as a documented engineering standard, not a project

**What success looks like:**  
No one asks whether to write a spec first. The question has been answered by the pipeline.

---

## Metrics That Matter

Standard DORA metrics (deployment frequency, lead time, change failure rate, MTTR) tell you whether velocity and quality are moving in the right direction. SDD adds two metrics specific to this process:

**Spec Coverage Ratio**  
The percentage of work items that have an associated, reviewed spec before implementation begins.

```
Spec Coverage = (tasks with linked spec) / (total tasks) × 100
```

Start tracking this from day one of the pilot. A rising spec coverage ratio is the leading indicator that the process is taking hold.

**Drift Rate**  
The percentage of completed features where the implementation diverged from the spec without a formal spec update.

```
Drift Rate = (features with undocumented divergence) / (total completed features) × 100
```

Drift is not always bad — requirements change. But undocumented drift means the spec is no longer the source of truth. Tracking it keeps the team honest.

---

## Handling Resistance

Two objections come up consistently. Both are worth addressing directly.

### "We bought the AI tool — why aren't we faster?"

This comes from management. The premise is that the tool alone delivers the speed gain.

The response: the tool provides potential. The process determines whether that potential is captured as productive output or wasted as rework. The speed gains in the research come from teams with disciplined processes, not from teams that simply have access to the tool. Show the metrics — spec coverage and drift rate — alongside DORA trends. Data beats feelings.

### "This is just more paperwork"

This comes from developers. The concern is that specs are overhead — documentation that slows you down.

The response: a spec is not documentation. Documentation is written after the fact to explain what was built. A spec is written before the fact to define what must be built. It drives the tests, the tasks, and the implementation. It is the production artifact, not the afterthought. The question to ask: would you rather spend two hours writing a clear spec, or two days debugging code that went off the rails because the intent was never clear?

---

## The Organizing Principle

Every phase, every metric, every objection comes back to one decision:

> We are making a conscious choice to shift our effort upstream.

More time invested in making intent explicit. Less time spent debugging output that missed the mark.

That is the trade SDD asks you to make. The 24-week plan is how you make it stick.
