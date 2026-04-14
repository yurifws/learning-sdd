# ORCHESTRATION.md

> Use this file when a feature is large enough to benefit from parallel specialist agents.
> For small features (1–3 tasks), skip this — sequential execution is safer and simpler.

---

## The Curse of Instructions

When you ask a single AI agent to handle an entire feature — database, API, UI, tests, security — you trigger what can be called the **curse of instructions**: the more you ask an agent to do in one prompt, the worse it gets at doing any of it correctly.

The cause is context overload. The agent is juggling too many concerns simultaneously. It starts making silent assumptions, mixing up layer responsibilities, forgetting earlier decisions, or producing code that's technically plausible but architecturally wrong.

This is not a model limitation you can fix with a better prompt. It is a structural problem. **The solution is structural**: break the work into small, focused tasks and give each agent only what it needs to do exactly one job.

```
Single agent + giant prompt:
  ✗ Forgets constraints set 20 exchanges ago
  ✗ Mixes business logic into persistence layer
  ✗ Produces code that works but violates the architecture
  ✗ Hard to review — every change touches everything

Specialist agents + focused tasks:
  ✓ Each agent sees only its own files and contract
  ✓ Clear boundaries prevent cross-layer contamination
  ✓ Each task is small enough to verify independently
  ✓ Failures are isolated — one agent's mistake doesn't cascade
```

The expert crew model is not about having more AIs. It is about giving each AI a job small enough to do correctly.

---

## What Is Orchestration

One **orchestrator agent** reads the full task list and delegates work to **specialist agents**.
Each specialist receives only the files relevant to its job — nothing more.

```
Orchestrator
├── reads: PROJECT_PLAN.md, PROJECT_ROADMAP.md, PROJECT_TECHSTACK.md, tasks.md
├── delegates Task A → Specialist 1  (sees: techstack + its own files only)
├── delegates Task B → Specialist 2  (sees: techstack + its own files only)
└── delegates Task C → Specialist 3  (sees: techstack + its own files only)
```

**Why smaller scope matters:** an agent that can only see its own files cannot accidentally
touch unrelated code, import wrong layers, or drift into out-of-scope changes.

---

## When to Orchestrate vs Execute Sequentially

| Use sequential (`implement-tasks`) | Use orchestration (`orchestrate-tasks`) |
|---|---|
| 1–3 tasks | 4+ tasks |
| Tasks share files | Tasks have clean file boundaries |
| High risk / sensitive domain | Well-understood, stable domain |
| First time implementing this pattern | Pattern already proven in codebase |

> Default to sequential. Orchestrate only when the task breakdown is truly atomic and parallel.

---

## Orchestrator Prompt Template

```
You are the orchestrator for [FEATURE NAME].

Read in order:
1. PROJECT_PLAN.md
2. PROJECT_ROADMAP.md
3. PROJECT_TECHSTACK.md
4. tasks.md

Your job:
- Delegate each task to a specialist agent
- Pass to each specialist: PROJECT_TECHSTACK.md + the specific files listed in the task
- Do NOT implement anything yourself
- After all specialists finish, verify: run [VERIFY COMMAND] and confirm it passes
- If any specialist reports a conflict or ambiguity, stop all work and flag [NEEDS_CLARIFICATION]
```

---

## Specialist Agent Prompt Template

```
You are a specialist agent responsible for one task only.

Your context:
- Stack rules: PROJECT_TECHSTACK.md
- Your task: [PASTE TASK BLOCK FROM tasks.md]
- Files you may touch: [LIST FROM TASK]

Rules:
- Touch ONLY the files listed above
- If you need a file not in your list, STOP and report it to the orchestrator
- Do not import from layers outside your boundary
- When done, run: [VERIFY COMMAND FOR THIS TASK] and confirm it passes
```

---

## Task Boundary Rules

For orchestration to work safely, each task in `tasks.md` must define:

```markdown
## Task N — [Name]
**Agent role:** [e.g. Persistence Specialist]
**Files to touch:** (list every file, no wildcards)
  - src/.../TaskEntity.java
  - src/.../TaskJpaRepository.java
  - src/.../TaskPersistenceService.java
**Must NOT touch:** (list boundaries)
  - Any controller or use case file
**Verify:** ./mvnw test -pl ... -Dtest=TaskPersistenceServiceTest
**Depends on:** Task N-1 must be merged before this starts
```

> If two tasks share even one file, they cannot run in parallel. Sequence them instead.

---

## Conflict Prevention Checklist

Before launching parallel agents, verify:

- [ ] Every task has an explicit `Files to touch` list
- [ ] No file appears in more than one parallel task
- [ ] Dependencies between tasks are sequenced (not parallelized)
- [ ] Each task has its own `Verify` command
- [ ] All `[NEEDS_CLARIFICATION]` items in `tasks.md` are resolved
