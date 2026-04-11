# SDD Examples

> Three complete examples showing SDD applied to real projects.
> Each is fully worked — not a template, not a sketch.

---

## Which Example Is Right for You

| Example | Architecture | Use it when |
|---|---|---|
| [`agent-os/`](agent-os/) | N/A — project brain pattern | You need persistent AI context across sessions (plan, roadmap, techstack) |
| [`java-hexagonal/`](java-hexagonal/) | Hexagonal (Ports & Adapters) | Complex domain, multiple adapters, long-lived codebase |
| [`java-layered/`](java-layered/) | Layered (Controller → Service → Repository) | CRUD API, simple domain, fast setup — also a GitHub-ready project template |

---

## Hexagonal vs. Layered — When to Choose

### Hexagonal Architecture (`java-hexagonal/`)

```
HTTP Request → Controller → Port In (interface)
                                  ↓
                            Use Case (domain logic)
                                  ↓
                            Port Out (interface)
                                  ↓
                            Persistence Service → Repository → DB
```

**Choose hexagonal when:**
- The domain has real business rules beyond CRUD
- You expect multiple input adapters (REST + events + batch jobs)
- You want the domain completely isolated from infrastructure
- The team is comfortable with more files/layers

**Trade-off:** More files, more ceremony. A simple CRUD feature crosses 6 layers.

---

### Layered Architecture (`java-layered/`)

```
HTTP Request → Controller → Service → Repository → DB
```

**Choose layered when:**
- The API is primarily CRUD with light business logic
- The team is small or new to the codebase
- Speed of delivery matters more than domain isolation
- You want to start simple and refactor later

**Trade-off:** Business logic tends to drift into the service layer over time.

---

## Agent-OS Pattern (`agent-os/`)

Not an architecture — a **session continuity pattern**.

Three files give any AI agent full project context instantly, without re-explaining across sessions:

| File | What it answers |
|---|---|
| `PROJECT_PLAN.md` | What are we building? What are we NOT building? |
| `PROJECT_ROADMAP.md` | What phase are we in? What's next? What's frozen? |
| `PROJECT_TECHSTACK.md` | What's the locked stack? What's banned? |

Use this pattern on top of either architecture.
