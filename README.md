# learning-sdd

A learning repository with four independent topics for building software with AI agents.
Each topic is self-contained — pick one and read it without needing the others.

---

## Start Here

**[`PROJECT_BOOTSTRAP.md`](PROJECT_BOOTSTRAP.md)** — Ready to start a real project? This is your step-by-step setup and feature workflow guide.

**[`QUICKSTART.md`](QUICKSTART.md)** — Not sure which topic to learn first? Read this. Takes 2 minutes.

---

## Topics

### [`sdd/`](sdd/) — Specification-Driven Development

A workflow where the spec is always the system of record. Every AI action traces back to a requirement. No guessing, no drift.

```
sdd/
├── README.md                    # What SDD is + reading order
├── WHY_SDD.md                   # Business case — ROI model, AI speed trap, adoption curve
├── ROLLOUT.md                   # 24-week adoption plan, maturity model, metrics, failure modes
├── PRODUCTION_RISKS.md          # Lethal trifecta (speed/non-determinism/cost), spec drift, AI security
├── EARS_REFERENCE.md            # How to write requirements the AI can follow
├── CLARIFICATION_GATE.md        # How to resolve all ambiguity before planning
├── LIVING_SPEC.md               # Spec-first discipline + commit convention
├── GUIDED_EXECUTION.md          # How to feed the AI one task at a time
├── TEST_FIRST_GATE.md           # How to enforce tests-before-code as governance
├── PROJECT_CONSTITUTION.md      # Non-negotiable rules template
├── TASKS.md                     # Atomic task list template
├── AGENTS.md                    # AI onboarding file template
├── SDD_REST_API_TEMPLATE.md     # Full API spec template with EARS
├── stacks/
│   ├── java-spring/             # Java 21 + Spring Boot — constitution + AI onboarding
│   └── react-nextjs/            # TypeScript + React — visual spec + AI onboarding
└── examples/
    ├── projects/                # Filled-in project templates (copy and adapt)
    │   ├── reference-project/   # Python + FastAPI — cleanest example of every template filled in
    │   ├── java-layered/        # Spring Boot, Layered Architecture
    │   └── java-hexagonal/      # Spring Boot, Hexagonal Architecture
    └── patterns/                # Architectural and governance patterns (reference illustrations)
        ├── agent-os/            # Persistent project brain + orchestration + contract-first workflow
        ├── architectural-drift/ # Drift taxonomy (4 types) + three-layer defense strategy
        ├── legacy-modernization/# Intent recovery + Strangler Fig incremental replacement
        ├── multi-agent/         # DB + API + UI + QA agents — full-stack orchestration walkthrough
        ├── multi-target/        # One spec → multiple language targets (invariants + generation pipeline)
        └── verification-gates/  # All four gates applied to a single feature end-to-end
```

---

### [`tds/`](tds/) — Test-Driven Specs

Turns specs from static documents into executable guardrails. Done is a verifiable fact, not an opinion.

```
tds/
├── README.md                    # What TDS is + reading order
├── SCENARIO_FORMAT.md           # Given/When/Then, 3 scenario types, translation rules
├── TEST_FIRST_GATE.md           # Governance checklist — tests must fail before code starts
├── PROPERTY_BASED_TESTING.md    # Proving behavior is always true (Hypothesis + FastCheck)
├── CHARACTERIZATION_TESTS.md    # Locking existing behavior before touching code with no spec
├── DEFINITION_OF_DONE.md        # Audit trail template, requirement→scenario→test mapping
└── example/                     # Full walkthrough: User Authentication feature
    ├── scenarios.md             # Happy path, edge cases, error cases
    ├── test-first-gate.md       # Filled gate checklist — all 14 tests confirmed failing
    ├── property-tests.md        # 4 property-based tests with Hypothesis (Python)
    └── audit-trail.md           # Complete audit trail: req ID → scenario → test → PR
```

---

### [`kiro/`](kiro/) — Spec-Driven IDE

An IDE that enforces a spec-driven workflow through steering files, agent hooks, and evidence generation.

```
kiro/
├── README.md                    # What Kiro is + core concepts
├── steering/
│   ├── techstack.md             # Locked stack, version pins, banned patterns
│   └── style.md                 # Naming, formatting, team conventions
├── hooks/
│   ├── HOOKS.md                 # Always / Ask / Never hook catalog
│   └── IMPLEMENTATION.md        # How to wire hooks (git, Claude Code, CI)
└── example-feature/             # Full execution spine: Expired Link feature
    ├── requirements.md          # EARS rules
    ├── design.md                # Architecture decisions
    ├── tasks.md                 # Atomic steps with Claude prompts
    ├── scenarios.md             # Given/When/Then scenarios (happy, edge, error)
    └── property-tests.md        # Tests derived from EARS clauses
```

---

### [`mcp/`](mcp/) — Model Context Protocol

An open protocol that gives AI agents controlled access to live systems for evidence-driven debugging.

```
mcp/
├── README.md                    # What MCP is + Runtime Gateflow + Spec-First Diagnostics
├── RUNBOOK.md                   # Fillable six-step diagnostic checklist
├── SETUP.md                     # How to install and configure MCP servers
├── servers/
│   ├── playwright.md            # Browser control — UI verification
│   ├── devtools.md              # Performance traces, network logs
│   └── figma.md                 # Design as source of truth
├── governance/
│   └── BOUNDED_AUTONOMY.md      # Always / Ask / Never rules for live-system access
└── example-diagnostic/          # Runtime Gateflow walkthroughs
    ├── spec-requirement.md      # Product list page — performance + empty state
    ├── evidence-log.md          # Observations tagged to spec clauses
    ├── root-cause.md            # Evidence mapped to file + line
    ├── fix-proposal.md          # Fixes + proof steps + spec updates
    ├── case-lcp/                # Dashboard LCP regression (spec: < 2.5s, actual: 4.1s)
    └── case-layout-thrashing/   # Janky animation — forced reflow in render loop
```
