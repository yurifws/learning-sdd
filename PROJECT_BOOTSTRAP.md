# Project Bootstrap — From Empty Repo to SDD-Ready

> This is the missing day-one guide. Everything you need exists in this repo — this file tells you what to do with it, in what order, before your first line of code.

---

## How to Use This Guide

There are two phases:

- **Phase 1 — Project setup** runs once, when you create the project. Takes about 30 minutes.
- **Phase 2 — Feature loop** runs for every feature you build. This is your repeatable workflow.

Each step links to the file that explains it in detail. Read the linked file if you need to understand why. Follow this checklist if you just need to move.

---

## Phase 1 — Project Setup (one time)

### Step 1 — Copy and fill the Constitution

The constitution is the non-negotiable rulebook for your project. Every AI agent and every developer reads it before touching anything.

```
Source:  sdd/PROJECT_CONSTITUTION.md
Action:  Copy to your repo root as CONSTITUTION.md. Fill in every section.
```

**Sections you must fill before writing any code:**

- [ ] **Project Overview** — one paragraph: what it is, what problem it solves
- [ ] **Architectural DNA** — architecture pattern, naming conventions, approved libraries
- [ ] **The Gates** — simplicity, anti-abstraction, security rules specific to your project
- [ ] **Do NOT List** — explicit list of things the AI must never do (modules it can't touch, patterns it can't introduce)
- [ ] **Out of Scope** — what this project is deliberately not doing

Leave API contracts (section 4.2) empty for now — those get filled per feature.

**If you are on Java/Spring Boot:** use [`sdd/stacks/java-spring/CONSTITUTION.md`](sdd/stacks/java-spring/CONSTITUTION.md) instead — it is pre-filled with layer rules, coding standards, and testing conventions for that stack.

**If you are on React/Next.js:** use [`sdd/stacks/react-nextjs/`](sdd/stacks/react-nextjs/) — it includes a visual spec methodology and pre-filled stack conventions.

Reference: [`sdd/PROJECT_CONSTITUTION.md`](sdd/PROJECT_CONSTITUTION.md)

---

### Step 2 — Write the AI onboarding file

The AI onboarding file is what the AI agent reads at the start of every session. It is the agent's persistent memory: project map, stack, commands, boundaries.

```
Source:  sdd/AGENTS.md
Action:  Copy to your repo root as AGENTS.md (or CLAUDE.md for Claude Code).
         Fill in every section.
```

**Sections to fill:**

- [ ] **Project map** — list of key files and what each one is for
- [ ] **Stack and commands** — how to build, run, and test the project
- [ ] **Boundaries** — what the AI is and is not allowed to do
- [ ] **Reading order** — which files the AI reads before starting a task

Reference: [`sdd/AGENTS.md`](sdd/AGENTS.md)

---

### Step 3 — Wire the enforcement hooks

Hooks turn your constitution rules into automated gates. They run in the editor, on commit, and in CI.

```
Source:  kiro/hooks/IMPLEMENTATION.md
Action:  Copy the relevant shell scripts to .githooks/. Wire CI via GitHub Actions.
```

**Minimum hooks to set up before first commit:**

- [ ] `NEVER-001` — block secrets from being committed (`.githooks/pre-commit`)
- [ ] `NEVER-002` — block `.env` files from being committed (`.githooks/pre-commit`)
- [ ] `NEVER-003` — block direct push to main (`.githooks/pre-push`)
- [ ] Register the hooks path: `git config core.hooksPath .githooks`

**Optional but recommended:**

- [ ] `ALWAYS-001` — run linter on every save (Claude Code `PostToolUse` hook)
- [ ] `ASK-002` — warn when the AI touches a file outside the current task scope

Reference: [`kiro/hooks/HOOKS.md`](kiro/hooks/HOOKS.md) · [`kiro/hooks/IMPLEMENTATION.md`](kiro/hooks/IMPLEMENTATION.md)

---

### Step 4 — Set up steering files

Steering files are long-term memory for the AI — facts it must know and rules it must follow in every session, independent of any specific task.

```
Source:  kiro/steering/techstack.md, kiro/steering/style.md
Action:  Copy both to a steering/ folder in your project. Fill in your stack and conventions.
```

- [ ] `techstack.md` — locked versions, banned libraries, required patterns
- [ ] `style.md` — naming conventions, formatting rules, team preferences

Reference: [`kiro/README.md` — Steering Files](kiro/README.md)

---

### Checkpoint — Ready to build

Before moving to Phase 2, confirm:

- [ ] `CONSTITUTION.md` filled and committed
- [ ] `AGENTS.md` (or `CLAUDE.md`) filled and committed
- [ ] `.githooks/` scripts in place and registered
- [ ] `steering/` files filled and committed
- [ ] First commit is clean — no secrets, no `.env`, no unreviewed dependencies

Your project is SDD-ready. Phase 2 starts with your first feature.

---

## Phase 2 — Feature Loop (every feature)

This is the workflow you run every time you build something new. Follow these steps in order. Do not skip ahead.

---

### Step 1 — Clarification Gate

Before writing a spec, surface every ambiguity. One unresolved question now costs ten times more after the spec is written.

```
Reference: sdd/CLARIFICATION_GATE.md
```

Go through the five question buckets:
- [ ] Behavior questions — what exactly does it do?
- [ ] Edge case questions — what happens at the boundaries?
- [ ] Error case questions — what fails and how?
- [ ] Security questions — who can access what?
- [ ] Integration questions — what does it depend on?

**Hard stops:**
- More than 5 open items → do not proceed. Schedule an alignment session.
- Any open item touches auth, PII, money, permissions, or deletion → full stop until resolved.

Flag every unresolved item as `[NEEDS_CLARIFICATION]` in the spec. Resolve all flags before the spec is locked.

---

### Step 2 — Write the spec

The spec is the system of record. Code follows the spec — never the other way around.

```
Source:  sdd/SDD_REST_API_TEMPLATE.md  (for REST APIs)
         sdd/EARS_REFERENCE.md         (for writing requirements)
Action:  Create specs/active/FEAT-XXX/requirements.md in your project.
         Write every requirement as an EARS clause.
```

EARS clause format:

```
WHEN [trigger],
  the system SHALL [outcome]
  AND SHALL NOT [forbidden behavior]
```

- [ ] Every requirement uses one of the five EARS patterns
- [ ] Every `SHALL` is specific enough to test
- [ ] Every `SHALL NOT` is explicit — nothing left implicit
- [ ] No open `[NEEDS_CLARIFICATION]` items remain

Reference: [`sdd/EARS_REFERENCE.md`](sdd/EARS_REFERENCE.md) · [`sdd/SDD_REST_API_TEMPLATE.md`](sdd/SDD_REST_API_TEMPLATE.md)

---

### Step 3 — Derive scenarios

Scenarios are the bridge between requirements and tests. Derive them directly from the EARS clauses.

```
Reference: tds/SCENARIO_FORMAT.md
Action:    Create specs/active/FEAT-XXX/scenarios.md.
           Every requirement gets at least one scenario of each type.
```

Three scenario types — all three are required:

| Type | What it covers |
|---|---|
| Happy path | Everything valid, system does what it should |
| Edge cases | Valid but boundary inputs — where most bugs hide |
| Error cases | Invalid inputs, missing auth, downstream failure |

- [ ] Every EARS clause maps to at least one scenario
- [ ] Every `Then` clause is a concrete, assertable outcome
- [ ] Every `does NOT` is an explicit negative assertion
- [ ] No scenario has more than one `When`

Reference: [`tds/SCENARIO_FORMAT.md`](tds/SCENARIO_FORMAT.md)

---

### Step 4 — Test-first gate

Write the tests. Confirm they fail. Do not write implementation until this gate passes.

```
Reference: tds/TEST_FIRST_GATE.md
Action:    Create tests from scenarios. Run them. Confirm every test fails.
           Fill in the test-first gate checklist before handing off to AI.
```

- [ ] Every scenario has a corresponding failing test
- [ ] Edge cases and error cases have tests — not just the happy path
- [ ] Every test has a verification command (`./mvnw test -Dtest=...`, `npx jest ...`)
- [ ] All tests are confirmed **failing** before any implementation is written
- [ ] Gate checklist is signed off

**The rule:** A test that passes before you write the implementation is not testing anything.

Reference: [`tds/TEST_FIRST_GATE.md`](tds/TEST_FIRST_GATE.md)

---

### Step 5 — Break into tasks

Decompose the spec into atomic tasks — one objective, one or two files, one PR each.

```
Source:  sdd/TASKS.md
Action:  Create specs/active/FEAT-XXX/tasks.md.
         Each task must be completable in a single AI session.
```

Each task must have:
- [ ] One clear objective
- [ ] The specific files it touches (and nothing else)
- [ ] The acceptance criterion — which scenario or test it satisfies
- [ ] A verification command

**The rule:** If a task references more than two files, split it. If it doesn't have an acceptance criterion from the spec, it is not ready.

Reference: [`sdd/TASKS.md`](sdd/TASKS.md)

---

### Step 6 — Feed the AI one task at a time

Give the AI one task. Verify it. Then give it the next. Never hand off multiple tasks in one session.

```
Reference: sdd/GUIDED_EXECUTION.md
```

For each task:
- [ ] Give the AI: the current task, the relevant spec clause, the relevant files — nothing more
- [ ] Run the acceptance criterion after every task
- [ ] If the AI touches a file not listed in the task → hard stop, investigate
- [ ] Do not move to the next task until the current one passes verification

**Drift detection:** If the AI's output diverges from the spec, stop. Do not patch. Update the spec first if the change was intentional, then continue.

Reference: [`sdd/GUIDED_EXECUTION.md`](sdd/GUIDED_EXECUTION.md)

---

### Step 7 — Verify and close

Before marking the feature done, run the full acceptance criteria and confirm the spec is still the system of record.

```
Reference: sdd/LIVING_SPEC.md · sdd/examples/verification-gates/RUNTIME_GATE.md
```

- [ ] All scenario tests pass
- [ ] All property-based tests pass (if written)
- [ ] The spec reflects the final behavior — no drift from implementation
- [ ] All tasks in `tasks.md` are checked off
- [ ] `AGENTS.md` is updated if the project map changed
- [ ] No unresolved `[NEEDS_CLARIFICATION]` items
- [ ] Audit trail complete: Requirement → Scenario → Test → PR

Reference: [`tds/DEFINITION_OF_DONE.md`](tds/DEFINITION_OF_DONE.md)

---

## When You Need Live Diagnostics

If a bug reaches staging or production and you need evidence from the live system:

```
Reference: mcp/RUNBOOK.md
Setup:     mcp/SETUP.md
```

1. Pick the EARS clause the bug violates
2. Reproduce using Playwright
3. Record evidence using DevTools
4. Map the evidence to file + line
5. Propose fix + verify against the spec clause
6. If the clause was vague, tighten it before closing

Reference: [`mcp/RUNBOOK.md`](mcp/RUNBOOK.md) · [`mcp/README.md`](mcp/README.md)

---

## For Legacy Systems

If you are applying SDD to a system that already exists with no spec:

**Before Phase 2:** Run the archaeological dig first.

1. Map current behavior and write characterization tests ([`tds/CHARACTERIZATION_TESTS.md`](tds/CHARACTERIZATION_TESTS.md))
2. Translate observations into spec candidates ([`sdd/examples/legacy-modernization/INTENT_RECOVERY.md`](sdd/examples/legacy-modernization/INTENT_RECOVERY.md))
3. Lock the spec — then begin Phase 2 for each new module
4. Replace incrementally using the Strangler Fig Pattern ([`sdd/examples/legacy-modernization/STRANGLER_FIG.md`](sdd/examples/legacy-modernization/STRANGLER_FIG.md))

---

## Quick Reference

| Phase | Step | Key file |
|---|---|---|
| Setup | Constitution | [`sdd/PROJECT_CONSTITUTION.md`](sdd/PROJECT_CONSTITUTION.md) |
| Setup | AI onboarding | [`sdd/AGENTS.md`](sdd/AGENTS.md) |
| Setup | Hooks | [`kiro/hooks/IMPLEMENTATION.md`](kiro/hooks/IMPLEMENTATION.md) |
| Setup | Steering | [`kiro/steering/techstack.md`](kiro/steering/techstack.md) |
| Feature | Clarification | [`sdd/CLARIFICATION_GATE.md`](sdd/CLARIFICATION_GATE.md) |
| Feature | Spec | [`sdd/SDD_REST_API_TEMPLATE.md`](sdd/SDD_REST_API_TEMPLATE.md) |
| Feature | Scenarios | [`tds/SCENARIO_FORMAT.md`](tds/SCENARIO_FORMAT.md) |
| Feature | Test-first gate | [`tds/TEST_FIRST_GATE.md`](tds/TEST_FIRST_GATE.md) |
| Feature | Tasks | [`sdd/TASKS.md`](sdd/TASKS.md) |
| Feature | AI execution | [`sdd/GUIDED_EXECUTION.md`](sdd/GUIDED_EXECUTION.md) |
| Feature | Done | [`tds/DEFINITION_OF_DONE.md`](tds/DEFINITION_OF_DONE.md) |
| Diagnostics | MCP runbook | [`mcp/RUNBOOK.md`](mcp/RUNBOOK.md) |
| Legacy | Intent recovery | [`sdd/examples/legacy-modernization/INTENT_RECOVERY.md`](sdd/examples/legacy-modernization/INTENT_RECOVERY.md) |
