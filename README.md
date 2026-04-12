# learning-sdd

A learning repository with three independent topics for building software with AI agents.
Each topic is self-contained — pick one and read it without needing the others.

---

## Start Here

**[`QUICKSTART.md`](QUICKSTART.md)** — Not sure which topic? Read this first. Takes 2 minutes.

---

## Topics

### [`sdd/`](sdd/) — Specification-Driven Development

A workflow where the spec is always the system of record. Every AI action traces back to a requirement. No guessing, no drift.

```
sdd/
├── README.md                    # What SDD is + reading order
├── EARS_REFERENCE.md            # How to write requirements the AI can follow
├── CLARIFICATION_GATE.md        # How to resolve all ambiguity before planning
├── LIVING_SPEC.md               # Spec-first discipline + commit convention
├── GUIDED_EXECUTION.md          # How to feed the AI one task at a time
├── PROJECT_CONSTITUTION.md      # Non-negotiable rules template
├── TASKS.md                     # Atomic task list template
├── AGENTS.md                    # AI onboarding file template
├── SDD_REST_API_TEMPLATE.md     # Full API spec template with EARS
├── stacks/
│   ├── java-spring/             # Java 21 + Spring Boot — constitution + AI onboarding
│   └── react-nextjs/            # TypeScript + React — visual spec + AI onboarding
└── examples/
    ├── agent-os/                # Persistent project brain pattern
    ├── java-hexagonal/          # Spring Boot, Hexagonal Architecture
    └── java-layered/            # Spring Boot, Layered Architecture (GitHub template)
```

---

### [`tds/`](tds/) — Test-Driven Specs

Turns specs from static documents into executable guardrails. Done is a verifiable fact, not an opinion.

```
tds/
├── README.md                    # What TDS is + reading order
├── SCENARIO_FORMAT.md           # Given/When/Then, 3 scenario types, translation rules
├── TEST_FIRST_GATE.md           # Governance checklist — tests must fail before code starts
├── PROPERTY_BASED_TESTING.md   # Proving behavior is always true (Hypothesis + FastCheck)
├── DEFINITION_OF_DONE.md       # Audit trail template, requirement→scenario→test mapping
└── example/                     # Full walkthrough: User Authentication feature
    ├── scenarios.md             # Happy path, edge cases, error cases
    ├── test-first-gate.md       # Filled gate checklist — all 17 tests confirmed failing
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
    └── property-tests.md        # Tests derived from EARS clauses
```

---

### [`mcp/`](mcp/) — Model Context Protocol

An open protocol that gives AI agents controlled access to live systems for evidence-driven debugging.

```
mcp/
├── README.md                    # What MCP is + Runtime Gateflow
├── SETUP.md                     # How to install and configure MCP servers
├── servers/
│   ├── playwright.md            # Browser control — UI verification
│   ├── devtools.md              # Performance traces, network logs
│   └── figma.md                 # Design as source of truth
├── governance/
│   └── BOUNDED_AUTONOMY.md      # Always / Ask / Never rules for live-system access
└── example-diagnostic/          # Runtime Gateflow walkthrough
    ├── spec-requirement.md      # Starting EARS clause
    ├── evidence-log.md          # Observations tagged to spec
    ├── root-cause.md            # Evidence mapped to file + line
    └── fix-proposal.md          # Fixes + proof steps + spec updates
```
