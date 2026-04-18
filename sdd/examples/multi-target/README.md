# Multi-Target Generation — Example

> What if the spec is the source code, and the code is just a compilation artifact?

---

## The Problem

You have a Go service. Performance requirements push the team to build a Rust version.

For two weeks, they are identical. Then a bug gets patched in Go but not Rust. A new validation rule lands in Go and gets forgotten in Rust. A security fix goes in one direction only.

Six months later, they are two distinct products. Maintaining them means double the work, double the mental overhead, and double the surface area for things to go wrong.

This is the drift that spec-driven development was built to prevent — but most teams only apply SDD to a single implementation. This example takes it further: **one spec, multiple targets**.

---

## The Mental Model Shift

The traditional model treats code as the primary asset:

```
Code (Go)  →  runs
Code (Rust) →  runs
           ↕ must be manually kept in sync
```

The spec-driven model treats the spec as the primary asset — and code as a compilation artifact:

```
Spec (one source of truth)
     ↓
Generation layer (acts like a compiler)
     ↓              ↓
 Go artifact    Rust artifact
```

Maintenance is no longer about patching two codebases. It is about updating the spec and regenerating. The spec is the real source code. Everything else is emitted.

---

## The LLVM Analogy

This is not a metaphor. It is the same engineering pattern at a higher level.

LLVM takes code written in any frontend language, converts it to a common IR (intermediate representation), and emits machine code for any target CPU. The IR is the single source of truth. The targets are interchangeable artifacts.

In spec-driven multi-target generation:

| LLVM concept | SDD equivalent |
|---|---|
| Frontend language (C, Rust, Swift) | The spec (EARS clauses, OpenAPI, contracts) |
| IR (intermediate representation) | The spec — same role |
| Backend target (x86, ARM, WASM) | Implementation target (Go, Rust, TypeScript, Flutter) |
| Compiled binary | Generated service or UI |

The spec is your IR. The implementations are your backends. Any target can be regenerated from the spec at any time.

---

## What This Example Covers

| File | What it teaches |
|---|---|
| [`INVARIANTS.md`](INVARIANTS.md) | The three classes of invariants — functional, contract, quality — that must hold identically across all targets, regardless of implementation |
| [`GENERATION_PIPELINE.md`](GENERATION_PIPELINE.md) | The four-step process for generating multiple implementations from one spec: language-agnostic spec → frozen API contract → per-target plans → shared benchmark harness |

---

## The Scenario

This example follows a **Payment Processing Service** — a financial-critical API that must run as both a primary Go service (general availability, existing infrastructure) and a Rust service (high-throughput tier, latency-sensitive workloads).

Key constraints:
- Both services must produce identical functional behavior for every input
- Both services must satisfy the same API contract — consumers must be able to switch targets without changing their integration
- The Rust service may use entirely different libraries, concurrency models, and runtime internals — as long as it satisfies the spec
- A single change to financial logic, error handling, or idempotency behavior must propagate to both targets or neither

---

## The One Number

When everything is stripped back, the entire model reduces to one number: **1**.

One spec. Not two codebases that drift. Not a spec plus some code that may or may not match it. One source of truth from which all targets are derived.

That is how divergence becomes structurally impossible instead of just undesirable.
