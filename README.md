# learning-sdd

A learning repository for **SDD (Specification-Driven Development)** — a methodology to control AI agents when building software. The goal is to give AI exactly what it needs, nothing more, and keep every change traceable back to a requirement.

---

## What is SDD?

SDD is a workflow where:
1. You define the **rules** before the AI touches any code (Constitution)
2. You surface and resolve **every ambiguity** before writing a single spec (Clarification Gate)
3. You define **what to build** with explicit contracts — spec always updated before code (Living Spec)
4. You break work into **atomic tasks** — one objective, 1–2 files each
5. You feed the AI **one task at a time** — guided execution, no drift
6. Every change is **verified** before it is accepted

---

## Repository Structure

```
learning-sdd/
│
│   ── Generic templates (stack-agnostic, copy & fill for any project)
├── PROJECT_CONSTITUTION.md        # Rulebook template: architecture, naming, security gates, do-not list
├── SDD_REST_API_TEMPLATE.md       # Design document template: endpoints, data models, EARS criteria
├── TASKS.md                       # Task list template: one objective per task, verify command, parallelism hints
├── GUIDED_EXECUTION.md            # AI execution rules: incremental disclosure, drift detection, Definition of Done
├── CLARIFICATION_GATE.md          # Ambiguity protocol: [NEEDS_CLARIFICATION] tag, thresholds, 5 buckets, spec-ready checklist
├── LIVING_SPEC.md                 # Spec-first discipline: 4-step rule, commit convention, Definition of Done
├── AGENTS.md                      # Template for writing an AGENTS.md for any project
│
│   ── Stack-specific templates (ready-to-use for Java/Spring Boot)
├── BACKEND_JAVA/
│   ├── PROJECT_CONSTITUTION_JAVA.md   # Constitution with Java/Spring Boot rules, testing, migrations
│   └── AGENTS_BACKEND_JAVA.md         # AI onboarding for hexagonal Spring Boot: layers, naming, conventions
│
│   ── Stack-specific templates (ready-to-use for TypeScript/React/Next.js)
├── FRONTEND/
│   ├── VISUAL_SPEC_TEMPLATE.md        # Visual spec template: tiers, pixel-to-requirement pipeline, parity checklist
│   └── AGENTS_FRONTEND.md             # AI onboarding for React/Next.js: visual spec protocol, tokens, states
│
│   ── Concrete example (filled-in SDD for a real Spring Boot API — Hexagonal Architecture)
├── example-api/
│   ├── CLAUDE.md                  # AI entrypoint: read these files in order before starting
│   ├── constitution.md            # Filled-in rules for this specific project
│   ├── spec.md                    # Full spec: hexagonal layers, endpoints, reference code for all 6 layers
│   ├── plan.md                    # Implementation progress and architectural decisions
│   └── task.md                    # Active task: what to implement right now
│
│   ── GitHub project template (ready-to-use Spring Boot SDD starter)
└── project-api/
    ├── CLAUDE.md                  # AI onboarding: reading order + post-task checklist
    ├── CONSTITUTION.md            # Global rules: layered architecture, DTOs, JWT, testing, error codes
    ├── CLARIFICATION_GATE.md      # Ambiguity protocol: tags, thresholds, 5 buckets, spec-ready checklist
    ├── LIVING_SPEC.md             # Spec-first discipline: 4-step rule, commit convention, Definition of Done
    ├── specs/
    │   ├── active/
    │   │   └── FEAT-XXX/
    │   │       ├── requirements.md      # EARS behavior rules (write before any code)
    │   │       ├── design.md            # Architecture, DTOs, endpoints, sequences
    │   │       ├── tasks.md             # Ordered tasks + Claude prompts
    │   │       └── lessons_learned.md  # Error log (check before debugging)
    │   └── archive/               # Completed features (move here after merge)
    └── .github/
        ├── PULL_REQUEST_TEMPLATE.md     # SPEC and IMPL PR checklists
        └── ISSUE_TEMPLATE/
            └── feature-spec.md          # GitHub issue template to kick off a feature
```

---

## The SDD Flow

```
Constitution       →  defines non-negotiable rules (architecture, naming, security)
     ↓
Clarification Gate →  surfaces every ambiguity before planning starts — [NEEDS_CLARIFICATION] tags, zero guessing
     ↓
SDD / Spec         →  defines what to build (endpoints, data models, success criteria)
     ↓
Tasks              →  breaks the spec into atomic steps (1 objective, 1-2 files, verify command)
     ↓
Guided AI          →  receives one task at a time, only the files listed
     ↓
Verify             →  acceptance criteria must pass before the task is marked done ✅
```

---

## Templates

### `example-api/` — Hexagonal Architecture Reference

A concrete **Spring Boot REST API** built with Hexagonal Architecture (Ports & Adapters). Shows SDD applied to a complex, multi-layer codebase.

**Tech Stack:** Java 17 + Spring Boot · Hexagonal Architecture · Spring Data JPA · MapStruct · Spring Security · OpenAPI 3

Every endpoint is implemented across 6 layers:
1. **Controller** — handles HTTP, no business logic
2. **OpenApi Interface** — Swagger contract + `@PreAuthorize`
3. **Port In** — input port interface
4. **Use Case** — business logic
5. **Port Out** — output port interface
6. **Persistence Service** — JPA repository calls

---

### `project-api/` — GitHub Project Template

A ready-to-use template for new projects. Uses a simpler **layered architecture** (Controller → Service → Repository) with a structured SPEC → IMPL PR workflow.

**Workflow:**
```
1.  Open GitHub Issue (use "New Feature Spec" template)
2.  Create branch: feat/FEAT-XXX-short-name
3.  Run CLARIFICATION_GATE.md — resolve all [NEEDS_CLARIFICATION] items
4.  Write specs/active/FEAT-XXX/requirements.md  ← EARS rules
5.  Write specs/active/FEAT-XXX/design.md        ← Architecture
6.  Open SPEC PR → team review → merge
7.  Write specs/active/FEAT-XXX/tasks.md         ← Task list
8.  Ask Claude to implement one task at a time
9.  Open IMPL PR → code review → merge
10. Archive: git mv specs/active/FEAT-XXX specs/archive/FEAT-XXX
```

**Branch & PR strategy:**

| Branch | PR Title | Contains |
|---|---|---|
| `feat/FEAT-XXX-short-name` | `[SPEC] FEAT-XXX Short Name` | spec files only |
| `feat/FEAT-XXX-short-name` | `[IMPL] FEAT-XXX Short Name` | Java code + tests |

> Rule: Spec PR must be approved before implementation starts.

---

## Key Principles

- **Spec drives code** — update the spec first, then the plan, then implement; never the other way around
- **Clarification before planning** — every ambiguity is flagged and resolved before any spec or code is written
- **Incremental disclosure** — give the AI only the files needed for the current task
- **One task = one PR** — if reviewing hurts your brain, the task was too big
- **Drift detection** — any file touched outside the task list is a hard stop
- **Audit trail** — every change traces back: Requirement → Task → PR → Acceptance Criteria
- **SPEC before IMPL** — requirements and design must be reviewed and approved before any code is written
