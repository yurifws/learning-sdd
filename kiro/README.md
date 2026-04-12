# kiro/

> An example package illustrating the core ideas behind **Amazon Kiro** — a spec-driven IDE that shifts the goal from _generating code_ to _generating evidence_.

---

## The Problem Kiro Solves

Standard AI tools dump code fast. No paper trail. No explanation of _why_. Ambiguity is buried in the output and surfaces later as a bug in production.

Kiro's answer: enforce a process that produces **durable artifacts** — files that prove the code you ended up with is the code you actually intended to build.

---

## Core Concepts

### 1. The Execution Spine

Three structured markdown files that are the backbone of every feature.
No code is written until all three exist and are approved.

| File | Question | Maps to SDD |
|---|---|---|
| `requirements.md` | What are we building? | EARS rules in `specs/active/FEAT-XXX/requirements.md` |
| `design.md` | How are we building it? | Architecture in `specs/active/FEAT-XXX/design.md` |
| `tasks.md` | What is the AI doing next? | Atomic steps in `specs/active/FEAT-XXX/tasks.md` |

---

### 2. EARS Requirements

**EARS** (Easy Approach to Requirements Syntax) forces every requirement to be:
- Unambiguous — no interpretable phrasing
- Precise — trigger + outcome, no filler
- Testable — directly derivable into test cases

**Vague (before EARS):**
> Handle expired links.

**EARS (after):**
```
WHEN the user clicks an expired link,
  the system SHALL display an HTTP 410 error page
  AND SHALL NOT redirect the user to any destination URL.
```

Five EARS patterns:

| Pattern | Keyword | Use when |
|---|---|---|
| Ubiquitous | _(none)_ | Always true — a rule that never has conditions |
| Event-driven | `WHEN` | A trigger causes a response |
| State-driven | `WHILE` | The system behaves differently in a given state |
| Conditional | `IF ... THEN` | A precondition guards the behavior |
| Unwanted behavior | `IF ... THEN SHALL NOT` | Explicitly forbidden paths |

---

### 3. Steering Files

A steering file is **long-term memory** for the AI agent. You write it once at project kickoff. The AI reads it before every session.

Two types:
- **`techstack.md`** — locked stack, version pins, banned libraries
- **`style.md`** — naming conventions, formatting rules, team preferences

> In SDD terms: steering files are the always-loaded complement to the Constitution. The Constitution defines rules the AI must follow; steering files define facts the AI must remember.

---

### 4. Agent Hooks

Hooks are automated rules tied to editor events (save, open, commit, etc.). They turn team standards from documentation nobody reads into enforcement nobody can skip.

Three tiers:

| Tier | Behavior | Example |
|---|---|---|
| **Always** | Runs automatically, no prompt | Run linter on every file save |
| **Ask** | Pauses and requests permission | "A new dependency is about to be added — approve?" |
| **Never** | Hard block — action is forbidden | Block commit if a file named `.env` is staged |

See [`hooks/HOOKS.md`](hooks/HOOKS.md) for full examples.

---

### 5. Evidence, Not Just Code

The real output of a Kiro workflow is a collection of artifacts:

```
requirements.md   →  proof of what was intended
design.md         →  proof of how it was planned
tasks.md          →  proof of what the AI was instructed to do
property-tests.md →  proof that it works under the full range of conditions
```

Property-based tests are generated directly from EARS requirements.
Each `WHEN/IF/THEN` clause becomes a family of test cases — not hand-picked examples,
but programmatically generated inputs that cover the full requirement boundary.

---

## Package Structure

```
kiro/
├── README.md                  ← you are here
├── steering/
│   ├── techstack.md           ← locked stack, banned patterns
│   └── style.md               ← naming, formatting, team rules
├── hooks/
│   ├── HOOKS.md               ← always / ask / never hook catalog
│   └── IMPLEMENTATION.md      ← how to wire hooks (git, Claude Code, CI)
└── example-feature/           ← full execution spine for "Expired Link" feature
    ├── requirements.md        ← EARS rules
    ├── design.md              ← architecture decisions
    ├── tasks.md               ← atomic steps for the AI
    ├── scenarios.md           ← Given/When/Then scenarios (happy, edge, error)
    └── property-tests.md      ← test families derived from requirements
```