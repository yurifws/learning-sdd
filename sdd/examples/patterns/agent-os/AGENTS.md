# AGENTS.md — AgentOS Context Bootstrap

> Read this file first. Then read the three files below in order before doing anything else.

---

## Purpose

This folder establishes the **persistent project brain** — the context that survives across every session, every agent, and every tool. You do not need to be re-briefed. You read these files and you are fully onboarded.

---

## Reading Order

Read these files **in this exact order** before touching any code, spec, or task:

1. [PROJECT_PLAN.md](PROJECT_PLAN.md) — what we are building and what we are NOT building
2. [PROJECT_ROADMAP.md](PROJECT_ROADMAP.md) — current phase and what comes next
3. [PROJECT_TECHSTACK.md](PROJECT_TECHSTACK.md) — locked stack decisions, banned patterns

---

## Rules

- **Never guess** what the project does — it is in `PROJECT_PLAN.md`.
- **Never assume** the priority of work — it is in `PROJECT_ROADMAP.md`.
- **Never substitute** libraries or patterns — they are locked in `PROJECT_TECHSTACK.md`.
- If any of these files have `[FILL]` placeholders, **stop and ask the human** to fill them before proceeding.
- If you want to change a decision in any of these files, **propose the change first** — do not just implement a different approach.

---

## Execution Mode

After reading the three context files, choose how to implement tasks:

| Mode | When to use |
|---|---|
| **Sequential** — one agent, one task at a time | Default. 1–3 tasks, or tasks that share files. |
| **Orchestration** — one orchestrator + specialist agents | 4+ tasks with clean file boundaries per task. |

If using orchestration, read [ORCHESTRATION.md](ORCHESTRATION.md) before delegating any work.

---

## When to Update These Files

| File | Update when... |
|---|---|
| `PROJECT_PLAN.md` | Scope changes, new "not building" decisions, success criteria change |
| `PROJECT_ROADMAP.md` | A phase completes, priorities shift, something gets frozen or unfrozen |
| `PROJECT_TECHSTACK.md` | A library is added, upgraded, or banned |
| `ORCHESTRATION.md` | A new feature needs orchestration — create one per feature, not one global file |

> Rule: update the file **before** implementing the change. These files are always the source of truth.