# SDD Examples

> Examples showing SDD applied to real projects and architectural patterns.  
> Organized into two categories: complete projects and concept patterns.

---

## 📁 [`projects/`](projects/) — Filled-in project templates

Fully worked SDD projects. Every template filled in for a real feature, on a real stack. Use these when you want to see what a complete spec-to-tasks workflow looks like in practice.

| Project | Stack | Architecture | Use it when |
|---------|-------|--------------|-------------|
| [`reference-project/`](projects/reference-project/) | Python · FastAPI | Layered | **Start here** — cleanest example of every template filled in |
| [`java-layered/`](projects/java-layered/) | Java · Spring Boot | Layered | CRUD API, simple domain, fast setup |
| [`java-hexagonal/`](projects/java-hexagonal/) | Java · Spring Boot | Hexagonal | Complex domain, multiple adapters, enterprise patterns |

---

## 📁 [`patterns/`](patterns/) — Architectural and governance patterns

Concept examples showing SDD applied to specific architectural challenges. Not copy-paste starting points — reference illustrations explaining *why* a practice exists.

| Pattern | What it covers | Use it when |
|---------|---------------|-------------|
| [`agent-os/`](patterns/agent-os/) | Session continuity brain files | You need persistent AI context across sessions |
| [`architectural-drift/`](patterns/architectural-drift/) | Four drift types · three defense layers | You want to catch AI agents drifting from your spec |
| [`legacy-modernization/`](patterns/legacy-modernization/) | Intent recovery · Strangler Fig | You need to modernize a legacy system safely |
| [`multi-agent/`](patterns/multi-agent/) | DB + API + UI + QA specialist agents | You want to coordinate multiple AI specialists |
| [`multi-target/`](patterns/multi-target/) | One spec → multiple implementations | You need one spec to generate code in different languages |
| [`verification-gates/`](patterns/verification-gates/) | SPEC · PLAN · TASK · RUNTIME gates | You want to see all four gates applied end-to-end |

---

## Hexagonal vs. Layered — When to Choose

### Layered (`projects/java-layered/`, `projects/reference-project/`)

```
HTTP Request → Controller → Service → Repository → DB
```

**Choose when:** CRUD API, simple domain, small team, speed matters.  
**Trade-off:** Business logic tends to drift into the service layer over time.

### Hexagonal (`projects/java-hexagonal/`)

```
HTTP Request → Controller → Port In (interface)
                                  ↓
                            Use Case (domain logic)
                                  ↓
                            Port Out (interface)
                                  ↓
                            Persistence Service → Repository → DB
```

**Choose when:** Complex domain, multiple input adapters (REST + gRPC + events), domain isolation matters.  
**Trade-off:** More files, more ceremony. A simple CRUD feature crosses 6 layers.
