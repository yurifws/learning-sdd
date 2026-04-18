# Project Examples

> Fully worked SDD projects — every template filled in for a real feature, on a real stack.  
> Use these when you want to see what a complete spec-to-tasks workflow looks like in practice.

---

## Examples

| Project | Stack | Architecture | Feature shown |
|---------|-------|--------------|---------------|
| [`reference-project/`](reference-project/) | Python · FastAPI · PostgreSQL | Layered | Send Notification (email/SMS) — the cleanest starting point |
| [`java-layered/`](java-layered/) | Java 17 · Spring Boot · PostgreSQL | Layered | Task CRUD with JWT auth |
| [`java-hexagonal/`](java-hexagonal/) | Java 17 · Spring Boot · H2/PostgreSQL | Hexagonal (Ports & Adapters) | Client management with gRPC adapter |

---

## How to Use These Examples

All three projects share the same folder skeleton:

```
<project>/
├── CONSTITUTION.md              ← non-negotiable rules for this project
├── AGENTS.md                    ← AI onboarding: commands, project map, conventions, boundaries
├── CLARIFICATION_GATE.md        ← all ambiguities resolved before the spec was written
└── specs/active/FEAT-XXX/
    ├── requirements.md          ← EARS requirements
    ├── design.md                ← architecture and data model
    ├── scenarios.md             ← Given/When/Then (reference-project only — see note)
    ├── TEST_FIRST_GATE.md       ← tests confirmed failing before implementation
    └── tasks.md                 ← ordered task list with verify commands
```

Reading order within any project:

1. `CONSTITUTION.md`
2. `AGENTS.md`
3. `CLARIFICATION_GATE.md`
4. `specs/active/FEAT-XXX/requirements.md`
5. `specs/active/FEAT-XXX/design.md`
6. `specs/active/FEAT-XXX/scenarios.md` *(reference-project only)*
7. `specs/active/FEAT-XXX/TEST_FIRST_GATE.md`
8. `specs/active/FEAT-XXX/tasks.md`

> **Note on `scenarios.md`:** only `reference-project/` includes a standalone scenarios file. The two Java examples embed scenario coverage directly in `requirements.md` and the failing-test evidence in `TEST_FIRST_GATE.md`. Both styles are valid — the standalone file is easier to review in a PR; the embedded style is faster for small features. Each project's own README calls out which style it uses.

---

## Which Example to Start With

**New to SDD?** → Start with [`reference-project/`](reference-project/). It uses the generic templates exactly as they appear in `/sdd`, filled in for a non-Java, non-TypeScript stack. Every file shows what the placeholder text becomes in a real project.

**Building a Spring Boot CRUD API?** → [`java-layered/`](java-layered/) is the fastest path. Copy the structure, swap the domain.

**Complex domain with multiple adapters?** → [`java-hexagonal/`](java-hexagonal/) shows Ports & Adapters applied end-to-end, including a gRPC input adapter alongside REST.

---

> For patterns and architectural concepts (drift detection, legacy modernization, multi-agent orchestration), see [`../patterns/`](../patterns/).
