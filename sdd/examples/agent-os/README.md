# agent-os — Persistent Project Brain

> The persistent project brain pattern keeps an AI agent oriented across every session.
> Instead of re-explaining the project each time, you write it down once — and every agent reads it first.

---

## The Problem It Solves

AI agents have no memory between sessions. Every new conversation starts cold.

Without a persistent brain:
- You paste context into every prompt and hope it's enough
- The agent drifts toward generic patterns instead of your team's patterns
- Stack decisions get re-litigated mid-task
- Scope creep happens silently because there's no documented "what we are NOT building"

With a persistent brain:
- The agent reads the project files before touching any code
- Stack choices, architecture rules, and scope are locked
- Every session starts from the same shared baseline

---

## What's in This Folder

| File | Purpose |
|---|---|
| [`PROJECT_PLAN.md`](PROJECT_PLAN.md) | Mission, scope, success criteria — the "what and why" |
| [`PROJECT_ROADMAP.md`](PROJECT_ROADMAP.md) | Milestones and phases — the "when" |
| [`PROJECT_TECHSTACK.md`](PROJECT_TECHSTACK.md) | Locked stack, version pins, banned patterns — the "how" |
| [`AGENTS.md`](AGENTS.md) | AI onboarding file — reading order + post-task checklist |
| [`ORCHESTRATION.md`](ORCHESTRATION.md) | How to delegate work across specialist agents in parallel |
| [`CONTRACT_FLOW.md`](CONTRACT_FLOW.md) | How agents hand off work to each other without breaking interfaces |
| [`example-java/`](example-java/) | A real project using this pattern (Java Spring Boot) |

---

## Reading Order

Start with the brain files if you're setting up a new project:

```
1. PROJECT_PLAN.md       ← fill this first — scope and mission
2. PROJECT_TECHSTACK.md  ← lock your stack before writing any code
3. PROJECT_ROADMAP.md    ← define milestones
4. AGENTS.md             ← write the AI onboarding file
```

Then, when a feature is large enough to split across multiple agents:

```
5. ORCHESTRATION.md      ← understand when and how to orchestrate
6. CONTRACT_FLOW.md      ← understand how agents hand off work safely
```

---

## Sequential vs. Orchestrated Execution

Not every feature needs orchestration. Use the right mode for the size of the job:

| Sequential (`implement-tasks`) | Orchestrated (`orchestrate-tasks`) |
|---|---|
| 1–3 tasks | 4+ tasks with clean boundaries |
| Tasks share files | Each task has its own file list |
| Sensitive or unfamiliar domain | Well-understood, stable domain |
| First time doing this | Pattern already proven in the codebase |

**Default to sequential.** Orchestration adds coordination overhead — only use it when tasks are truly independent and parallel.
