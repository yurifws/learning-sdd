# multi-agent — Full-Stack Orchestration Example

> A concrete walkthrough of four specialist agents building a feature end-to-end.
> **Feature:** Task Dashboard — list and create tasks via a web UI
> **Stack:** PostgreSQL · Node.js/Express · React/TypeScript · Jest + Cypress

---

## The Four Specialists

| Agent | Owns | Produces |
|---|---|---|
| **DB Agent** | Schema, migrations, data access layer | `db-contract.md` |
| **API Agent** | REST endpoints, validation, error handling | `api-contract.md` |
| **UI Agent** | React components, API calls, state | `ui-contract.md` |
| **QA Agent** | Unit tests, integration tests, E2E tests | Test coverage report |

Each agent sees only the contracts from upstream layers. The UI Agent never reads database files. The QA Agent reads all three contracts and verifies everything connects.

---

## Files in This Folder

| File | What it is |
|---|---|
| `README.md` | This file — overview and reading guide |
| `requirements.md` | EARS requirements for the Task Dashboard feature |
| `orchestration-run.md` | **Full walkthrough** — exact prompts, agent responses, gate checks |
| `db-contract.md` | Contract produced by DB Agent — locked at Gate B |
| `api-contract.md` | Contract produced by API Agent — locked at Gate C |
| `ui-contract.md` | Contract produced by UI Agent — locked at Gate D |

---

## Reading Order

```
1. requirements.md       ← what the system must do (EARS rules)
2. orchestration-run.md  ← how the four agents build it (read this as the main example)
3. db-contract.md        ← the DB layer contract (referenced in the walkthrough)
4. api-contract.md       ← the API layer contract (referenced in the walkthrough)
5. ui-contract.md        ← the UI layer contract (referenced in the walkthrough)
```
