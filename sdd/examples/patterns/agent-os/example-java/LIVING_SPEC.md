# LIVING_SPEC.md — Spec-First Discipline

> The spec is always the system of record.
> Code reflects the spec. If they diverge, the spec wins — update the code, not the spec.

---

## The 4-Step Rule

Before writing any code:

```
1. Update requirements.md  — WHAT changes (EARS clause added, modified, or removed)
2. Update design.md        — HOW it changes (new endpoint, DTO field, exception)
3. Update tasks.md         — WHAT to implement (atomic task added to the list)
4. Implement               — Only now write code
```

If you skip step 1, 2, or 3 and go straight to code: **stop, go back**.

---

## Commit Convention

Every change follows this 3-commit pattern:

```
spec: add EARS clause for soft delete (§2.6)
plan: add TASK-05 — implement TaskUseCase.softDelete
impl: implement soft delete in TaskUseCase and TaskPersistenceService
```

Mixing spec + code in one commit is not allowed. A reviewer must be able to read the spec commit before reading the implementation commit.

---

## When to Update Each File

| File | Update when... |
|---|---|
| `requirements.md` | A requirement is added, changed, or removed |
| `design.md` | A new endpoint, DTO field, exception type, or architectural decision is added |
| `tasks.md` | A new task is added, or an existing task's scope changes |
| `CLARIFICATION_GATE.md` | A new ambiguity is resolved — add the Q&A before implementing |
| `CONSTITUTION.md` | A new non-negotiable rule is agreed on by the team |

---

## What "Approved" Means

A requirement in `requirements.md` is **Approved** when:
1. The clarification gate has been run for this requirement (all 5 buckets answered)
2. All `[NEEDS_CLARIFICATION]` items are resolved
3. A team member (or the spec author) has signed off

Do not implement any requirement that is not Approved.

---

## Definition of Done

A task is done when ALL of the following are true:

- [ ] The requirement in `requirements.md` is Approved
- [ ] The test was written BEFORE implementation (see `TEST_FIRST_GATE.md`)
- [ ] The test was confirmed failing before any code was written
- [ ] The implementation makes the test pass
- [ ] `./mvnw test` passes — zero failures
- [ ] Coverage ≥ 80% on the changed service class
- [ ] `design.md` reflects the final behavior — no drift from implementation
- [ ] `tasks.md` task is checked off (`[x]`)
- [ ] No unresolved `[NEEDS_CLARIFICATION]` items in any file
- [ ] No unresolved TODOs or FIXMEs in changed files

---

## Spec Drift Detection

Run this check before every PR:

```bash
# Find all [NEEDS_CLARIFICATION] items
grep -r "\[NEEDS_CLARIFICATION\]" .

# Find all TODO/FIXME in implementation files
grep -r "TODO\|FIXME" src/

# Find unresolved [FILL] placeholders
grep -r "\[FILL\]" .
```

Any result = the PR is not ready.

---

## Audit Trail

Every implemented requirement must trace back through:

```
Requirement (requirements.md §X.Y)
    ↓
Design decision (design.md §Z)
    ↓
Task (tasks.md TASK-NN)
    ↓
Test (TaskUseCaseTest.java or TaskControllerTest.java)
    ↓
PR (GitHub PR #N)
```

If any link is missing, add it before closing the PR.
