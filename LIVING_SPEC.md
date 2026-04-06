# Living Spec

> Your documentation should drive code, not describe it.
> Every decision, constraint, and change lives in the repo — version-controlled, reviewable, and auditable.

---

## The Problem with Static Docs

Documentation in Confluence or Google Docs is frozen in time. It becomes obsolete
the moment code diverges from it — which happens almost immediately.
The result is a reactive loop: update the doc to match the code you already built.

A living spec flips this: **update the spec first, then the code follows.**

---

## What Lives in the Repo

| File | Purpose |
|---|---|
| `spec.md` / `design.md` | Intent and constraints — what to build and why |
| `plan.md` | Architecture and key decisions |
| `tasks.md` | Ordered implementation steps |
| `CLAUDE.md` / `AGENTS.md` | AI instruction files — persistent context for agents |

These files are not documentation after the fact. They are the system of record.

---

## The 4-Step Golden Rule

> Follow this order every time something changes. No exceptions.

```
1. Update the spec     ← change the intent (what + why)
2. Update the plan     ← change the architecture and tasks (how)
3. Implement           ← write the code, mapped to tasks
4. Verify              ← confirm code aligns with the updated spec
```

Never write code before step 1 is done. Never merge before step 4 passes.

---

## Commit Convention

| Commit | Contains | Message format |
|---|---|---|
| 1st | Spec update only | `spec: update intent for [feature]` |
| 2nd | Plan + tasks update | `plan: update architecture for [feature]` |
| 3+ | Implementation | `feat: [task-id] short description` |

This makes your Git history a log of decisions, not just code changes.

---

## Why This Matters for AI Agents

AI agents have an **attention-budget problem**: on long tasks, context gets diluted
and the agent forgets constraints set earlier, falling back to generic defaults.

Persistent, version-controlled files solve this. An agent can re-read `CLAUDE.md`,
`spec.md`, or `CONSTITUTION.md` at any point and reload the full project mental model.

> Chat history is temporary. Files in Git are permanent.

This is why `CLAUDE.md` must always be kept current — it is the agent's persistent memory.

---

## Definition of Done

A feature is complete only when every box is checked:

- [ ] Spec (`spec.md` / `design.md`) reflects the final behavior — no drift from implementation
- [ ] Plan (`plan.md`) updated with final architecture and any decisions made during implementation
- [ ] All tasks in `tasks.md` checked off
- [ ] AI instruction files (`CLAUDE.md` / `AGENTS.md`) are current
- [ ] All acceptance criteria pass (automated + manual)
- [ ] No unresolved `[NEEDS_CLARIFICATION]` items
- [ ] Zero unresolved TODOs or FIXMEs in changed files
- [ ] PR merged and linked to task IDs

---

## Audit Trail

When every intent change is a commit, your Git history becomes:
- A log of decisions and their rationale
- Evidence for compliance frameworks (ISO, SOC 2, etc.)
- A safe map of intent for refactoring legacy systems

```
spec commit  (why)
  └── plan commit  (how)
        └── implementation commits  (what)
              └── PR → merged, traceable to spec
```
