# PROJECT_TECHSTACK.md

> Fill this once at project kickoff. These are locked decisions — no guessing, no substitutions without updating this file first.

---

## Language & Runtime

| Concern | Decision | Version |
|---|---|---|
| Language | [FILL: e.g. Java] | [FILL: e.g. 17] |
| Runtime / Framework | [FILL: e.g. Spring Boot] | [FILL: e.g. 3.2] |
| Build tool | [FILL: e.g. Maven] | [FILL] |

---

## Key Libraries

| Role | Library | Version | Notes |
|---|---|---|---|
| Database access | [FILL] | [FILL] | [FILL] |
| Auth | [FILL] | [FILL] | [FILL] |
| Testing | [FILL] | [FILL] | [FILL] |
| [Other] | [FILL] | [FILL] | [FILL] |

---

## Architecture Pattern

<!-- One line. The AI must follow this — no alternatives. -->
[FILL: e.g. "Hexagonal Architecture (Ports & Adapters). Controller → Port In → Use Case → Port Out → Persistence."]

---

## Banned / Off-limits

<!-- Libraries or patterns we explicitly do not use. -->
- [FILL: e.g. "No Lombok — use records or plain classes."]
- [FILL: e.g. "No field injection (@Autowired on fields) — constructor injection only."]
- [FILL]

---

## Infrastructure

| Concern | Decision |
|---|---|
| Database | [FILL: e.g. PostgreSQL 15] |
| Auth mechanism | [FILL: e.g. JWT, RS256] |
| Deployment target | [FILL: e.g. Docker + Railway] |
| CI | [FILL: e.g. GitHub Actions] |