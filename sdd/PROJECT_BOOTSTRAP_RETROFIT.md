# SDD Retrofit — Adopting SDD on an Existing Production Project

> **Who this is for:** Teams with a running codebase that want to start using SDD without rewriting history.  
> **What you need:** A codebase Claude can read. Nothing else.  
> **Time to first governed feature:** ~45 minutes.

---

## The Core Idea

You don't need to document the past. SDD governs **new changes from today forward**.  
The two files you need before any new work begins are:

1. **`CONSTITUTION.md`** — the non-negotiable rules already implicit in your codebase
2. **`AGENTS.md`** — the project map your AI reads before touching anything

Once those two files exist and are committed, you run the standard feature loop for every new change.

---

## Phase 1 — Generate the Foundation Files (one-time, ~45 min)

### Step 1 — Generate `CONSTITUTION.md`

Open a conversation with Claude and paste this prompt, filling in the bracketed parts:

```
You are helping me adopt SDD (Specification-Driven Development) on an existing project.

Read the codebase at [path or paste key files / describe the project if you can't share code].

Then generate a filled-in CONSTITUTION.md using the template at sdd/PROJECT_CONSTITUTION.md.

Fill every section with real values inferred from the code — do not use placeholder text:

- Section 1 (Project Overview): what this system does and the problem it solves, in plain language
- Section 2 (Architectural DNA): the actual architecture pattern (layered, hexagonal, monolith,
  microservices, event-driven — whatever is there), real naming conventions inferred from actual
  symbol names, real folder structure, and the actual dependency list found in the manifest file
  (package.json, pom.xml, build.gradle, pyproject.toml, go.mod, Gemfile, Cargo.toml, etc.)
- Section 3 (The Gates): mark any gate rule already violated in the codebase as [KNOWN EXCEPTION]
  with a one-line explanation — do NOT silently drop a rule just because the code doesn't follow it yet
- Section 4 (Technical Blueprint): the real data model entities and their relationships (infer from
  ORM models, schema files, migration files, or DB layer code), the real auth/security approach
  (JWT, session, OAuth, API key, none — whatever is there), the real deployment pattern if visible
- Section 6 (Do NOT List): infer what is clearly shared, fragile, or load-bearing from the code
  structure — these are the things the AI must never touch without explicit instruction
- Section 8 (Clarification Gate): flag anything you cannot infer with [NEEDS_CLARIFICATION] —
  never guess, never silently omit

Do NOT invent rules that don't exist in the codebase.
Do NOT drop template sections just because the code doesn't follow them — mark gaps honestly.
```

Review the output carefully. Every `[NEEDS_CLARIFICATION]` item is a real decision you must make — fill them in before committing.

---

### Step 2 — Generate `AGENTS.md`

In the same conversation (so Claude still has codebase context), paste:

```
Now generate a filled-in AGENTS.md using the template at sdd/AGENTS.md.

Fill every section with real values from the codebase — do not use placeholder text:

- Section 2 (Commands): the actual build, test, lint, format, and start commands — look in
  package.json scripts, Makefile, pom.xml, build.gradle, pyproject.toml, or equivalent.
  If a command isn't defined, write [NOT CONFIGURED] — do not guess
- Section 3 (Project Map): the real folder tree (top two levels), with a one-line description
  for each directory — infer purpose from file names and contents, not from conventions
- Section 4 (Stack & Conventions): naming conventions inferred from actual file names and
  symbol names in the code — not generic defaults. If conventions are inconsistent, note it
- Section 8 (Architecture Rules): infer the actual layer rules from the import graph and class
  structure — flag anything inconsistent across the codebase as [INCONSISTENCY FOUND: ...]

Mark anything you cannot infer confidently as [NEEDS_CLARIFICATION].
```

---

### Step 3 — Commit

```bash
git checkout -b chore/adopt-sdd
# Place both files at the root of the project (alongside src/, tests/, etc.)
git add CONSTITUTION.md AGENTS.md
git commit -m "chore: adopt SDD — add CONSTITUTION.md and AGENTS.md"
git push -u origin chore/adopt-sdd
# Open a PR — have a teammate review both files before merging
```

> **Why a PR?** Both files codify implicit team knowledge. A review surfaces disagreements before the AI acts on wrong assumptions.

---

## Phase 2 — First Feature Under SDD

Once the foundation files are merged, every new feature follows the standard loop.

```
Clarification Gate → Requirements (EARS) → Design → Scenarios → Test-First Gate → Tasks → Guided Execution → Verify
```

For the first feature, use this prompt to kick off the Clarification Gate:

```
Read CONSTITUTION.md and AGENTS.md.

I want to implement: [one-sentence description of the feature]

Before writing any spec, run the Clarification Gate from sdd/CLARIFICATION_GATE.md.
List every question you have. Do not guess. Do not start any spec until I answer them.
```

Then follow the standard SDD feature loop documented in `sdd/README.md`.

---

## What to Do with Existing Code

| Situation | What to do |
|-----------|-----------|
| Old code has no tests | Note it in `CONSTITUTION.md` Section 6 as a known gap. New code must be tested. |
| Old code violates your new architecture rules | Mark it `[KNOWN EXCEPTION]` in CONSTITUTION.md. Don't fix it retroactively — fix it when you touch it next. |
| Old code has implicit contracts (undocumented APIs, events, queues) | Document them in `specs/active/LEGACY/` as a low-priority spec. Do not let AI change them without a spec. |
| You find inconsistencies while generating files | Surface them as `[INCONSISTENCY FOUND]` in AGENTS.md. Resolve in the PR review. |
| No manifest file (scripts, notebooks, legacy) | Describe the runtime and dependencies in plain text in Section 2 — the template works for any project type. |

---

## Choosing the Right Starting Template

If a stack-specific template exists for your technology, use it — it comes pre-filled with conventions.  
If not, the generic template works for any language, framework, or project type.

| Your stack | Template to use |
|------------|----------------|
| Java + Spring Boot | [`stacks/java-spring/CONSTITUTION.md`](stacks/java-spring/CONSTITUTION.md) + [`stacks/java-spring/AGENTS.md`](stacks/java-spring/AGENTS.md) |
| TypeScript + React / Next.js | [`stacks/react-nextjs/CONSTITUTION.md`](stacks/react-nextjs/CONSTITUTION.md) + [`stacks/react-nextjs/AGENTS.md`](stacks/react-nextjs/AGENTS.md) |
| Any other stack | [`PROJECT_CONSTITUTION.md`](PROJECT_CONSTITUTION.md) + [`AGENTS.md`](AGENTS.md) |

> The generic templates cover backend, frontend, mobile, data pipelines, CLIs, and monorepos.  
> Stack-specific templates are shortcuts — they are not required.

For a complete worked example of filled-in files, see [`examples/projects/reference-project/`](examples/projects/reference-project/).

---

## Checklist — Ready to Start SDD?

- [ ] `CONSTITUTION.md` committed — no open `[NEEDS_CLARIFICATION]` items
- [ ] `AGENTS.md` committed — commands tested and confirmed working
- [ ] PR reviewed by at least one teammate
- [ ] First feature has a Clarification Gate open
- [ ] Team knows: spec first, code second — always

> Once this checklist is done, follow `sdd/README.md` reading order for the feature loop.
