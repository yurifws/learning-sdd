# QUICKSTART.md

> Three independent topics. Pick one. You don't need to read the others.

---

## Topic 1 — SDD (Specification-Driven Development)

**What it is:** A workflow for building software with AI agents. Spec always comes before code. Every change is traceable back to a requirement.

**Start here:** [`sdd/README.md`](sdd/README.md)

**Good for you if:**
- You want a repeatable, structured process for working with AI
- You're starting a new project and want to avoid drift and technical debt
- You want to know how to write requirements the AI can actually follow

---

## Topic 2 — Kiro (Spec-Driven IDE)

**What it is:** An IDE that enforces a spec-driven workflow through steering files, agent hooks, and evidence generation. Turns team standards into automated rules nobody can skip.

**Start here:** [`kiro/README.md`](kiro/README.md)

**Good for you if:**
- You want to automate enforcement of coding standards at the editor level
- You want to understand EARS requirements and the execution spine pattern
- You want AI to generate evidence (tests, artifacts) that prove the code matches the spec

---

## Topic 3 — MCP (Model Context Protocol)

**What it is:** An open protocol that gives AI agents controlled access to live systems — browsers, DevTools, design files. Turns AI from a guesser into an evidence-driven diagnostic partner.

**Start here:** [`mcp/README.md`](mcp/README.md)

**Good for you if:**
- You want AI to debug using real runtime data, not static code inference
- You want to connect AI to Playwright, Chrome DevTools, or Figma
- You want a structured diagnostic loop anchored to your spec

---

## What's in each topic

```
sdd/        ← Core methodology: concepts, templates, stack guides, examples
kiro/       ← Spec-driven IDE: steering files, hooks, execution spine, property tests
mcp/        ← Live-system observation: servers, governance, diagnostic walkthrough
```

Each folder is self-contained. No topic assumes you've read another.
