# SDD — Specification-Driven Development

> A workflow for building software with AI agents where the spec is always the system of record, every change is traceable back to a requirement, and the AI is fed exactly what it needs — nothing more.

---

## The Problem SDD Solves

AI coding assistants are fast but undisciplined. Left without structure, they:
- Generate code without a clear trail of why
- Drift away from requirements mid-task
- Bury ambiguity in output that surfaces as a bug weeks later

SDD fixes this by making the AI follow a strict process before writing a single line of code.

---

## The SDD Workflow

```
Constitution       →  defines non-negotiable rules (architecture, naming, security)
     ↓
Clarification Gate →  surfaces every ambiguity before planning — zero guessing
     ↓
Living Spec        →  spec updated first, then plan, then code — spec is always current
     ↓
Tasks              →  breaks the spec into atomic steps (1 objective, 1–2 files each)
     ↓
Test-First Gate    →  every task has tests written and confirmed failing before AI touches code
     ↓
Guided AI          →  receives one task at a time, only the files it needs
     ↓
Verify             →  acceptance criteria must pass before the task is marked done ✅
```

---

## Reading Order

Read in this order. Each file builds on the previous.

| Step | File | What it answers |
|---|---|---|
| 1 | [`EARS_REFERENCE.md`](EARS_REFERENCE.md) | How to write requirements the AI can follow |
| 2 | [`CLARIFICATION_GATE.md`](CLARIFICATION_GATE.md) | How to surface and resolve all ambiguity before writing any spec |
| 3 | [`LIVING_SPEC.md`](LIVING_SPEC.md) | How to keep the spec as the system of record |
| 4 | [`GUIDED_EXECUTION.md`](GUIDED_EXECUTION.md) | How to feed the AI one task at a time without drift |
| 5 | [`TEST_FIRST_GATE.md`](TEST_FIRST_GATE.md) | How to enforce tests-before-code as governance |
| 6 | [`PROJECT_CONSTITUTION.md`](PROJECT_CONSTITUTION.md) | How to define non-negotiable rules the AI always follows |
| 7 | [`TASKS.md`](TASKS.md) | How to break a spec into atomic, verifiable steps |
| 8 | [`AGENTS.md`](AGENTS.md) | How to write the onboarding file your AI reads first |

For a complete API spec template: [`SDD_REST_API_TEMPLATE.md`](SDD_REST_API_TEMPLATE.md)

---

## Choose Your Stack

Stack-specific implementations of SDD with pre-filled templates and conventions:

- [`stacks/java-spring/`](stacks/java-spring/) — Java 21 + Spring Boot, Hexagonal or Layered architecture
- [`stacks/react-nextjs/`](stacks/react-nextjs/) — TypeScript + React/Next.js, visual spec methodology

---

## See It in Action

Complete worked examples showing SDD applied to real projects:

- [`examples/`](examples/) — Agent-OS brain (with orchestration + contract-first workflow), Java Hexagonal API, Java Layered API (GitHub-ready template), Verification Gates end-to-end walkthrough, Multi-Agent full-stack example (DB + API + UI + QA)

---

## Key Principles

- **Spec drives code** — update the spec first, then the plan, then implement
- **Clarification before planning** — every ambiguity flagged and resolved before any spec is written
- **Incremental disclosure** — give the AI only the files needed for the current task
- **Drift detection** — any file touched outside the task list is a hard stop
- **Audit trail** — every change traces back: Requirement → Task → PR → Acceptance Criteria
