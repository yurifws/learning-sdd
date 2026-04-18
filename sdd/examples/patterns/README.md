# Pattern Examples

> Conceptual examples showing SDD applied to architectural challenges.  
> These are not project templates — they are worked illustrations of specific patterns.  
> Use these when you want to understand a concept before applying it to your project.

---

## Examples

| Pattern | What it covers | Use it when |
|---------|---------------|-------------|
| [`agent-os/`](agent-os/) | Session continuity — PROJECT_PLAN, PROJECT_ROADMAP, PROJECT_TECHSTACK brain files | You need persistent AI context across sessions |
| [`architectural-drift/`](architectural-drift/) | Four drift types · three defense layers · Order Service scenario | You want to catch and prevent AI agents drifting from your spec |
| [`legacy-modernization/`](legacy-modernization/) | Intent recovery · Strangler Fig Pattern · validation tiers | You need to replace a legacy system without losing the intent buried in the old code |
| [`multi-agent/`](multi-agent/) | DB + API + UI + QA specialist agents coordinating via contracts | You want to coordinate multiple AI specialists on a single feature |
| [`multi-target/`](multi-target/) | One spec → multiple implementations · invariant classes · determinism engineering | You need one spec to generate functionally identical code in different languages |
| [`verification-gates/`](verification-gates/) | SPEC_GATE · PLAN_GATE · TASK_GATE · RUNTIME_GATE applied end-to-end | You want to see all four gates applied to a single feature |

---

## How These Differ from Project Examples

Project examples ([`../projects/`](../projects/)) show complete, filled-in templates for a working codebase. You copy them and adapt.

Pattern examples show **concepts in action** — they explain *why* a practice exists and demonstrate it with a realistic scenario. They are not copy-paste starting points; they are reference illustrations.

---

## Reading Order Within Each Pattern

Each pattern folder contains:
- `README.md` — overview, when to use, quick reference
- One or more concept files (e.g., `DRIFT_TAXONOMY.md`, `INTENT_RECOVERY.md`) — the worked illustration

Start with the `README.md` in the pattern folder. It explains the concept and tells you which files to read in which order.
