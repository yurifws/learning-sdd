# mcp/

> An example package illustrating the **Model Context Protocol (MCP)** — an open protocol that connects AI agents to live systems, turning them from guessers into evidence-driven diagnostic partners.

---

## The Core Problem

When you ask an AI to fix a bug, it works from static code and pasted logs. It has no idea what's happening at runtime. The result is speculation — plausible-looking changes that may fix nothing, or introduce new bugs.

MCP solves this by giving the AI a **controlled bridge** to live tools: browsers, DevTools, design files. Instead of inferring what's broken, the agent can observe it directly and show you proof.

---

## Speculation vs. Observation

| Speculation (without MCP) | Observation (with MCP) |
|---|---|
| AI infers from static code | AI observes live runtime behavior |
| Evidence: pasted logs and code snippets | Evidence: captured traces, screenshots, network logs |
| Output: plausible guess | Output: verifiable proof linked to spec |
| Debugging improves code | Debugging improves spec + code |

---

## What Is MCP?

MCP (Model Context Protocol) is an **open protocol** that standardizes how AI hosts (your IDE, your agent) connect to external tools and data sources.

Key properties:

- **Open** — any tool can implement the protocol; no vendor lock-in
- **Controlled** — the AI can only call tools you have explicitly declared and permitted
- **Auditable** — every tool call is visible and traceable
- **Standard** — uses a client-host-server architecture over JSON-RPC

```
┌──────────────────────┐         JSON-RPC         ┌─────────────────────┐
│   AI Host (IDE /     │◄────────────────────────►│   MCP Server        │
│   Agent)             │    tool declarations +    │   (Playwright /     │
│                      │    structured responses   │   DevTools / Figma) │
└──────────────────────┘                           └─────────────────────┘
```

---

## Bounded Autonomy

The most important safety concept in MCP. The agent is **not** free to roam your system.

> The agent can only call the specific tools that you have explicitly declared and given it permission to use.

This is what makes MCP safe for real projects. See [`governance/BOUNDED_AUTONOMY.md`](governance/BOUNDED_AUTONOMY.md) for the full three-tier rule set.

---

## The MCP Server Toolkit

Three specialized servers cover the most common diagnostic scenarios:

| Server | Layer | What it gives the agent |
|---|---|---|
| **Playwright** | Frontend / UX | Browser control — navigate, click, type, capture screenshots |
| **Chrome DevTools** | Runtime / Performance | Performance traces, network logs, JS execution timelines |
| **Figma** | Design | Live access to design artifacts as the source of truth |

See the [`servers/`](servers/) directory for a full breakdown of each.

---

## The Runtime Gateflow

A structured, repeatable diagnostic loop that plugs into a spec-driven workflow:

```
1. Pick requirement   →  select one EARS clause from the spec as the starting point
       ↓
2. Reproduce          →  use Playwright to re-enact the user journey exactly as specified
       ↓
3. Record evidence    →  use DevTools to capture traces, logs, and network activity
       ↓
4. Localize           →  agent maps evidence to the root cause (file, line, function)
       ↓
5. Propose fix        →  targeted change + proof steps to verify it resolves the requirement
       ↓
6. Update spec        →  if the evidence reveals the requirement was vague, improve it
```

Every step is anchored to the spec. Evidence never floats free — it is always linked back to the requirement that triggered the investigation.

See [`example-diagnostic/`](example-diagnostic/) for a full walkthrough.

---

## Evidence Anchored to Spec

This is the feedback loop that makes MCP more than just a debugging tool.

When the agent gathers evidence and finds the root cause, it doesn't just fix code. It asks: **was the requirement clear enough to prevent this?**

If the answer is no — if a vague requirement like "the page should be fast" made it impossible to detect the violation earlier — the spec gets updated with a precise, testable EARS clause.

```
Vague spec   →  Evidence reveals ambiguity  →  Precise spec
"page should be fast"                         "WHEN the product list loads,
                                               the system SHALL respond within
                                               1,500ms at p95 on a standard connection."
```

Over time, every debugging session makes the spec stronger.

---

## How This Relates to SDD

| SDD concept | MCP equivalent |
|---|---|
| EARS requirements | Starting point for every Runtime Gateflow run |
| Acceptance criteria | The test the agent must prove passes before closing |
| Clarification Gate | Evidence loop — ambiguity is caught by runtime observation |
| Living Spec | Spec updated when evidence reveals a gap |
| Hooks (Always/Ask/Never) | Bounded autonomy governance over live-system access |

The core SDD principle — **spec is the system of record** — extends here to runtime behavior. The spec doesn't just define what to build; it defines what to observe and what counts as proof.
