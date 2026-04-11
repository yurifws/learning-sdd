# Living Spec

> Your documentation should drive code, not describe it.
> Every decision, constraint, and change lives in the repo — version-controlled, reviewable, and auditable.
>
> Before writing any spec: apply `CLARIFICATION_GATE.md` — resolve all ambiguity first.

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
| `specs/active/FEAT-XXX/design.md` | Intent and constraints — what to build and why |
| `specs/active/FEAT-XXX/requirements.md` | EARS behavior rules |
| `specs/active/FEAT-XXX/tasks.md` | Ordered implementation steps |
| `specs/active/FEAT-XXX/lessons_learned.md` | Decisions made during implementation |
| `CLAUDE.md` | AI instruction file — persistent context for agents |

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
`design.md`, or `CONSTITUTION.md` at any point and reload the full project mental model.

> Chat history is temporary. Files in Git are permanent.

This is why `CLAUDE.md` must always be kept current — it is the agent's persistent memory.

---

## Definition of Done

A feature is complete only when every box is checked:

- [ ] `design.md` reflects the final behavior — no drift from implementation
- [ ] `lessons_learned.md` updated with any decisions made during implementation
- [ ] All tasks in `tasks.md` checked off
- [ ] `CLAUDE.md` is current
- [ ] All acceptance criteria in `requirements.md` Section 6 pass
- [ ] No unresolved `[NEEDS_CLARIFICATION]` items
- [ ] `./mvnw test` passes — zero failures, coverage ≥ 80%
- [ ] Zero unresolved TODOs or FIXMEs in changed files
- [ ] IMPL PR merged and linked to feature ID

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
              └── IMPL PR → merged, traceable to FEAT-XXX
```
