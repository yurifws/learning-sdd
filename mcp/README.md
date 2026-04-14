# mcp/

> An example package illustrating the **Model Context Protocol (MCP)** — an open protocol that connects AI agents to live systems, turning them from guessers into evidence-driven diagnostic partners.

---

## The Core Problem

When you ask an AI to fix a bug, it works from static code and pasted logs. It has no idea what's happening at runtime. The result is speculation — plausible-looking changes that may fix nothing, or introduce new bugs.

MCP solves this by giving the AI a **controlled bridge** to live tools: browsers, DevTools, design files. Instead of inferring what's broken, the agent can observe it directly and show you proof.

---

## Spec-First Diagnostics

The single most important discipline in MCP-based debugging.

> **Before looking at code, check the spec. The spec defines what "correct" looks like.**

Without a spec anchor, a debugging session drifts. The agent explores code, forms hypotheses, proposes changes — but has no definition of done. The session ends when something *seems* better, not when the requirement is *demonstrably met*.

Spec-first diagnostics reverses this. Every investigation begins with a specific EARS clause:

```
WHEN the user navigates to /dashboard,
  the system SHALL achieve an LCP below 2.5 seconds
  as measured by Web Vitals on a standard broadband connection.
```

This one clause answers four questions before a single tool is called:

| Question | Answer from the spec |
|---|---|
| What are we measuring? | LCP (Largest Contentful Paint) |
| What is the threshold? | 2.5 seconds |
| Under what conditions? | Standard broadband |
| How will we know it's fixed? | Web Vitals reports < 2.5s |

The agent never has to guess what "fixed" means. The requirement defines it.

### The Feedback Loop

Debugging sessions improve the spec, not just the code:

```
Vague report      →  Pick a spec clause   →  Gather evidence
"feels slow"            (or write one)         (trace, logs, screenshots)
                              ↑                        ↓
                    Update spec if vague  ←  Root cause localized
                    "SHALL < 2.5s LCP"        (file, line, function)
                              ↑                        ↓
                         Spec stronger    ←  Fix verified against clause
```

Over time, every bug that reaches production tightens a requirement that was too vague to catch it earlier. The spec becomes a living record of what the system must always do.

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

Every observation in `evidence-log.md` must cite the spec clause it relates to. Evidence that isn't linked to a requirement is noise — it has no definition of done.

When the agent gathers evidence and finds the root cause, it doesn't just fix code. It asks: **was the requirement clear enough to prevent this?**

If the answer is no — if a vague requirement like "the page should be fast" made it impossible to detect the violation earlier — the spec gets updated with a precise, testable EARS clause:

```
Vague spec   →  Evidence reveals ambiguity  →  Precise spec
"page should be fast"                         "WHEN the product list loads,
                                               the system SHALL respond within
                                               1,500ms at p95 on a standard connection."
```

**The rule:** close every diagnostic session with either a passing requirement or a sharper spec. Never close with "seems better."

---

## MCP + Property Tests

Property-based testing finds the minimal input that breaks a rule. MCP tells you exactly what the system does when that input arrives.

When a property test fails and shrinking produces a counterexample, the natural next step is to observe the failure in the live system — not guess from the stack trace. The Runtime Gateflow handles this directly:

```
Property test fails
  → shrinking produces minimal counterexample (e.g. quantity=0, token="a b")
         ↓
1. Pick requirement   →  the EARS clause the failing property was derived from
       ↓
2. Reproduce          →  use Playwright to send the exact counterexample as a live request
       ↓
3. Record evidence    →  use DevTools to capture what the system actually returned
                         (status code, response body, DB state, network log)
       ↓
4. Localize           →  agent maps the observed behavior to the file and line that handled it
       ↓
5. Propose fix        →  targeted change + re-run the property test to confirm it passes
       ↓
6. Update spec        →  if the EARS clause was ambiguous enough to allow the bug,
                         tighten it to prevent the same class of failure
```

**Why this matters:** A shrunk counterexample is already the minimal case — you don't need to simplify the reproduction. Feed it straight into Playwright and let the agent observe the live behavior. The evidence log answers: what did the system *actually* do, versus what the property said it must *always* do.

For property-based testing fundamentals and framework setup (Hypothesis, FastCheck, jqwik), see [`tds/PROPERTY_BASED_TESTING.md`](../tds/PROPERTY_BASED_TESTING.md).

---

## Examples

```
example-diagnostic/
├── spec-requirement.md      # Product list page — performance + empty state
├── evidence-log.md          # 4 pieces of evidence tagged to spec clauses
├── root-cause.md            # 4 root causes mapped to file + line
├── fix-proposal.md          # 3 targeted fixes + 2 spec updates
├── case-lcp/                # Dashboard LCP regression (spec: < 2.5s, actual: 4.1s)
│   ├── spec-requirement.md
│   ├── evidence-log.md
│   ├── root-cause.md
│   └── fix-proposal.md
└── case-layout-thrashing/   # Janky animation — forced reflow in render loop
    ├── spec-requirement.md
    ├── evidence-log.md
    ├── root-cause.md
    └── fix-proposal.md
```

For a repeatable, fillable version of the six-step process: [`RUNBOOK.md`](RUNBOOK.md)