# AI in Production — Risks and Defenses

> The gap between fast and reliable. What breaks when you ship vibe code, and how to defend against it.  
> For the business case, see [`WHY_SDD.md`](WHY_SDD.md). For the rollout plan, see [`ROLLOUT.md`](ROLLOUT.md).

---

## The Fork in the Road

There are two fundamentally different questions you can ask about a piece of code:

- *Can we get this to work?* — the prototyping mindset
- *Can we prove this will keep working?* — the engineering mindset

Both questions are valid. The prototyping mindset is exactly right for exploration and discovery. The engineering mindset is required for production.

The danger is when code that was generated under the prototyping mindset gets shipped to production without making the shift. The result looks fine on the surface, passes basic tests, and then collapses under real-world pressure — because the AI was never given a solid spec. Without a spec, it invented the business logic, the error handling, and the security model. It had no choice but to guess.

---

## The Lethal Trifecta

Production failures from AI-generated code almost always trace back to the same three forces combining. Individually, each one looks like a benefit. Together, they create a perfect storm.

### 1. Speed

AI generates code orders of magnitude faster than humans can review it. The review queue becomes a firehose. Engineers get overwhelmed and start approving PRs just to clear the backlog. Quality degrades not because people stopped caring, but because the volume made careful review structurally impossible.

**Defense: the micro PR rule.**  
One task, one PR. Each PR must be small enough that a reviewer can fully understand it in a few minutes. Verification is part of the task definition — not something added at the end. Large tasks get decomposed before implementation begins, never after.

The goal is to keep human review feasible at AI generation speed.

### 2. Non-Determinism

AI models are probabilistic. The same prompt can produce different code on different runs. This breaks a fundamental assumption most developers carry: that given the same inputs, the process produces the same outputs.

The practical consequences are severe. "It worked on my machine" no longer means anything. Automated pipelines can fail non-deterministically. A feature that worked in development may behave differently in staging for no reproducible reason.

**Defense: enforce reproducibility.**  
On the technical side: lower temperature settings reduce randomness; seeds can make outputs repeatable. But the deeper fix is cultural. Stop trusting what the AI produces and start proving it. Every piece of AI-generated code must be validated against the spec with automated tests. The spec is the fixed reference — the AI output is the variable that must be verified against it.

### 3. Cost

When generating code feels free, the temptation to skip the expensive parts — careful design, thorough tests, security review — becomes overwhelming. The logic feels intuitive: if code is cheap, why pay the overhead?

The flaw is in what "cost" is being measured. The cost of generating code is near zero. The cost of maintaining, debugging, and fixing code that was never designed or tested correctly is enormous. A 3am production incident, weeks of debugging, and lost customer trust are all downstream costs of skipping governance upstream.

**Defense: make governance non-negotiable.**  
Not as suggestions in a wiki. As automated gates in the pipeline. Code that is missing tests, contains hardcoded secrets, or skips authentication checks cannot be merged — the pipeline blocks it. Done is not "it works." Done is "it works, all tests pass, and every behavior traces back to a spec clause."

---

## Two Traps That Compound the Trifecta

### Trap 1: The Vague Prompt

When you give an AI a vague instruction — "implement a new UI flow," "add user management" — it has no choice but to invent the details. It will guess your business rules, your error handling, your security model. The guess will be generic and almost certainly wrong for your specific context.

The fix is to replace prompts with spec contracts. Instead of describing what you want in conversational terms, write a precise requirement:

```
WHEN a user submits the login form with valid credentials,
  the system SHALL redirect to the dashboard
  AND SHALL set a session token with a 24-hour expiry.
WHEN a user submits with invalid credentials,
  the system SHALL return an error message
  AND SHALL NOT reveal whether the username or password was wrong.
```

No ambiguity means no guessing. The AI has an exact target.

### Trap 2: Spec Drift

In traditional development, code is the source of truth. When you make a fix, you change the code.

When AI is part of your workflow, the spec is the source of truth. If you make a code fix without updating the spec, the next time the AI touches that feature it reads the old spec — and reintroduces the exact bug you just fixed.

Spec drift is particularly dangerous because it compounds. Each undocumented divergence makes the spec less reliable as a reference, which makes future AI output less reliable, which creates more bugs, which create more "quick fixes" that never get documented.

**The rule:** any behavioral change in the code requires a spec update first. Code and spec stay synchronized — the spec always describes what the system actually does, not what it was supposed to do three sprints ago.

---

## The Three-Step Defense

These are not suggestions. They are the minimum disciplines required to ship AI-generated code safely.

**1. Spec diff first.**  
Before any code changes, the spec changes. If the behavior is changing, the requirement that governs that behavior is updated and reviewed first. Code is written to satisfy the updated spec, not the other way around.

**2. Enforce plan mode.**  
The AI must show its plan before generating code. The plan is reviewed and approved by a human. Code generation does not begin until the plan is signed off. This catches misaligned intent before it becomes misaligned implementation.

**3. Automated validation gates.**  
Every merge requires: tests present and passing, no hardcoded secrets, no missing authentication on protected endpoints, spec reference present. These checks are automated and enforced in CI — they cannot be bypassed by individual developers under deadline pressure.

---

## Security: AI Agents as an Attack Surface

When an AI agent has tool access — reading files, browsing URLs, calling APIs — it becomes a new attack vector. This is not theoretical.

**Prompt injection** is the most common risk. Malicious content in external sources (a webpage the agent browsed, a file it read, a response from an API) can contain instructions that hijack the agent's behavior. The agent cannot distinguish between your instructions and instructions embedded in data it reads.

Defenses:

- **Allow lists over block lists.** Define exactly which tools, domains, and APIs the agent is permitted to access. Anything not on the list is denied. Block lists are always incomplete.
- **Treat external data as hostile.** Every piece of information the agent retrieves from outside your system — web pages, external APIs, user-submitted content — must be treated as untrusted until proven otherwise. It should never be executed as instruction.
- **Minimal permissions.** The agent should have the minimum access required for the current task. An agent writing tests does not need production database credentials. Scope access to the task, not the system.
- **Human approval for irreversible actions.** Any action that cannot be undone — deleting records, sending emails, deploying to production — requires explicit human confirmation before the agent executes it.

---

## The Question That Matters

The choice between building a solid engineering collaborator and shipping a fragile pile of fast code comes down to one thing: whether you treat these disciplines as optional or as the default.

The tools are not the constraint. The discipline is.
