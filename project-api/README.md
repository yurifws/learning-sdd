# Spring Boot SDD Template

A GitHub project template for building Java Spring Boot APIs using **Specification-Driven Development (SDD)** with Claude.

---

## How this works

```
CONSTITUTION.md                ← Global rules. Claude always reads this first.
specs/
  ├── active/
  │   └── FEAT-XXX/
  │       ├── requirements.md      ← EARS behavior rules (write before any code)
  │       ├── design.md            ← Architecture, DTOs, endpoints, sequences
  │       ├── tasks.md             ← Ordered tasks + Claude prompts
  │       └── lessons_learned.md  ← Error log (check before debugging)
  └── archive/                 ← Completed features (move here after merge)
src/                           ← Java Spring Boot code (not included — initialize with Spring Initializr)
.github/                       ← PR template + issue templates
CLAUDE.md                      ← AI entrypoint: reading order + post-task checklist
CLARIFICATION_GATE.md          ← Mandatory ambiguity protocol: tags, thresholds, 5 buckets, spec-ready checklist
```

---

## Workflow

```
1. Open GitHub Issue (use "New Feature Spec" template)
2. Create branch: feat/FEAT-XXX-short-name
3. Write specs/active/FEAT-XXX/requirements.md  ← EARS rules
4. Write specs/active/FEAT-XXX/design.md        ← Architecture
5. Open SPEC PR → team review → merge
6. Write specs/active/FEAT-XXX/tasks.md         ← Task list
7. Ask Claude to implement one task at a time
8. Open IMPL PR → code review → merge
9. Archive: git mv specs/active/FEAT-XXX specs/archive/FEAT-XXX
```

---

## Kickoff Prompt for Claude

Copy this into Claude to start any feature:

```
I'm building a Java Spring Boot API.
Read CONSTITUTION.md and all files in specs/active/FEAT-XXX/ before starting.
Follow CONSTITUTION.md at all times — no exceptions.

Start with TASK-01 from tasks.md.
After each task, tell me what you built and wait for my approval.
Log any errors in lessons_learned.md.
```

---

## Branch & PR Strategy

| Branch | PR Title | Contains |
|---|---|---|
| `feat/FEAT-XXX-short-name` | `[SPEC] FEAT-XXX Short Name` | spec files only |
| `feat/FEAT-XXX-short-name` | `[IMPL] FEAT-XXX Short Name` | Java code + tests |

**Rule:** Spec PR must be merged and approved before implementation starts.

---

## EARS Quick Reference

| Pattern | Template |
|---|---|
| Always | `The [system] SHALL [behavior]` |
| Event | `WHEN [event], the [system] SHALL [action]` |
| State | `WHILE [state], the [system] SHALL [behavior]` |
| Optional | `WHERE [feature enabled], the [system] SHALL [behavior]` |
| Unwanted | `IF [bad input], THEN SHALL [error] AND SHALL NOT [harm]` |

---

## Resources

- [Anthropic Prompting Guide](https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/overview)
- [GitHub Spec Kit](https://github.com/github/spec-kit)
- [Thoughtworks — SDD in 2025](https://www.thoughtworks.com/en-us/insights/blog/agile-engineering-practices/spec-driven-development-unpacking-2025-new-engineering-practices)
