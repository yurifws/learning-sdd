# Legacy Modernization — Example

> Maybe that terrifying legacy system isn't just a mountain of technical debt. Maybe it's a rich dataset of proven, battle-tested business logic — waiting for you to recover its intent and build its future.

---

## The Problem

The system has been running for ten years. It works. Nobody knows why.

The original developers left. The documentation was last updated in 2017. There are functions with names like `calcFinalAmtV3_FIXED_new` and `legacyInvoiceFlow_DO_NOT_TOUCH`. Every time someone proposes a rewrite, the conversation ends the same way: "We can't risk it. Last time we touched that module, we broke billing for three regions."

So the system stays. Tech debt compounds. Feature velocity crawls. Everyone knows it has to change. Nobody wants to be the one who breaks it.

---

## The Modernization Trap

Most rewrite attempts fail for the same reason: they focus entirely on the **how** — the code, the architecture, the new framework — without ever recovering the **what** and the **why**.

The result is a new system that looks better, passes a few demos, and then quietly misbehaves in production because it missed a dozen implicit business rules nobody knew to document.

This is called **intent decay**: the original purpose of the software — the reason that weird conditional exists, the reason that calculation has an edge case for accounts created before 2014 — evaporates over time. What remains is only code. And code without intent is just behavior without explanation.

When intent is gone, teams are stuck maintaining by patching: another workaround, another special case, another piece of duct tape. Nobody can make fundamental improvements because nobody can prove what fundamental means for this system.

---

## The SDD Approach

Spec-driven development treats modernization as a two-phase problem.

**Phase 1 — Recover the intent.** Before writing a single line of new code, run an archaeological dig. Map current behavior. Lock it with characterization tests. Translate it into a structured spec. The spec becomes the system of record — not the old code, not someone's memory, not a Confluence page.

**Phase 2 — Replace incrementally.** With intent recovered and written down, use the Strangler Fig Pattern to build new services around the old system. Reroute traffic piece by piece. Validate conformance at every step. Shut the old system down only when every behavior in the spec is proven to exist in the new one.

---

## What This Example Covers

| File | What it teaches |
|---|---|
| [`INTENT_RECOVERY.md`](INTENT_RECOVERY.md) | Phase 1 — the archaeological dig: characterization tests, spec structure (invariants, scenarios, do-not list), and how AI agents assist the recovery |
| [`STRANGLER_FIG.md`](STRANGLER_FIG.md) | Phase 2 — incremental modernization: the Strangler Fig Pattern, the AI-assisted rebuild pipeline, and why conformance — not completion — is the definition of done |

---

## The Scenario

This example follows a **Billing Service** — a ten-year-old Java monolith handling invoice generation, proration, tax calculation, and payment reconciliation for a SaaS product with 200,000 accounts.

Key facts:
- Three of the five original engineers have left the company
- The last architectural diagram is from 2019 and doesn't match the code
- There are 47 open bugs, 12 of which are labeled "investigate — do not close"
- Tax calculation has a known edge case for enterprise accounts that nobody has been able to replicate consistently
- Every rewrite attempt in the last four years has been abandoned

The business has decided: this time, it happens. The question is how to do it without destroying billing for 200,000 accounts.
