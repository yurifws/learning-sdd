# Architectural Drift — Example

> Your system is drifting right now. The only real question is: is that drift being caught, made visible, and managed — or is it quietly rotting from the inside out?

---

## The Problem

You wrote a spec. You shipped the code.

Six months later, the frontend team filed a bug saying the API was returning `OUT_OF_STOCK` instead of `ITEM_OUT_OF_STOCK`. Nobody changed it intentionally — it just drifted. Someone refactored the error constants under pressure, no spec was updated, no gate caught it.

That is the smallest version of drift.

The dangerous version is when it compounds: fields renamed, behaviors changed, endpoints added in the shadows, and a growing pile of undocumented decisions. The spec becomes a lie. The blueprints show a fire escape; reality has a brick wall.

---

## What Architectural Drift Is

**Architectural drift** is the gap between your system's intended design — what lives in the spec — and what it actually does in production.

It is not a documentation problem. It is a reliability, security, and predictability problem.

It compounds silently. There is no single breaking moment. There is just a slow, irreversible separation between the system you think you're running and the one that actually is.

---

## What This Example Covers

| File | What it teaches |
|---|---|
| [`DRIFT_TAXONOMY.md`](DRIFT_TAXONOMY.md) | The four kinds of drift — structural, behavioral, security, evolutionary — with concrete HTTP examples from the Order Service |
| [`DEFENSE_LAYERS.md`](DEFENSE_LAYERS.md) | The three-layer defense strategy — pre-merge, pre-deploy, runtime — and what each catches before it reaches production |

---

## The Scenario

This example follows a single system: an **Order Service API** used by a web frontend, a mobile app, and an inventory backend. The API was specced out at launch 18 months ago. Since then:

- Three teams have made changes
- Two major releases shipped under deadline pressure
- Documentation reviews were always deferred to "next sprint"

The spec still says version 1. The system is running something closer to version 3.

This example names exactly what drifted, how each type of drift was created, and how a defense-in-depth pipeline would have caught it before it reached production.

---

## The Core Shift

In the classical way, code is the only truth. The spec is a launch artifact. Drift is inevitable because there is no mechanism to prevent it.

In the spec-first way, the spec is the system of record. The code's job is to continuously prove it conforms. Drift is not just avoided — it is caught automatically and immediately.

```
Classical:  [Spec written once] → [Code evolves freely] → [Drift accumulates] → [Incident]

Spec-first: [Spec is locked]    → [Code must prove conformance] → [Drift caught at gate] → [Spec updated deliberately]
```

The unit of delivery is no longer just code. It is **the spec plus the validators that prove the code matches it**.

The code can be refactored, rewritten, or regenerated — as long as it keeps passing those checks.
