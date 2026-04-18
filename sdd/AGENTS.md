# AGENTS.md — AI Onboarding Packet
# [Project Name]

> This file is written FOR the AI agent, not for humans.
> Read this entirely before writing or modifying any code.
> If anything in this file conflicts with another file, this file wins — flag the discrepancy.

---

## 1. Project Overview

**Project:** [What this system is and what problem it solves — one paragraph]

**Architecture:** [e.g., Layered MVC / Hexagonal / Microservices / Monolith]

**Language & Runtime:** [e.g., Java 21 / Python 3.12 / Go 1.22 / TypeScript + Node 20]

**Framework:** [e.g., Spring Boot / FastAPI / Gin / Next.js]

---

## 2. Commands

| Task | Command |
|---|---|
| Install dependencies | `[e.g., pnpm install / pip install -r requirements.txt / go mod tidy]` |
| Start dev server | `[e.g., pnpm dev / uvicorn main:app --reload / go run ./cmd/server]` |
| Run all tests | `[e.g., pnpm test / pytest / go test ./... / ./mvnw test]` |
| Run single test | `[e.g., pnpm test path/to/file / pytest tests/test_foo.py / ./mvnw test -Dtest=FooTest]` |
| Lint | `[e.g., pnpm lint / ruff check . / golangci-lint run]` |
| Format code | `[e.g., pnpm format / black . / gofmt -w .]` |
| Type check | `[e.g., pnpm typecheck / mypy . / tsc --noEmit]` |
| Build / package | `[e.g., pnpm build / ./mvnw package / go build ./...]` |

> Always use the project's own tooling wrapper. Never use a globally installed version of the build tool.

---

## 3. Project Map

```
[your-project-root]/
├── [source directory]       # Main application code
│   ├── [domain/feature A]   # [brief description]
│   └── [domain/feature B]   # [brief description]
├── [test directory]         # All test files — mirror the source structure
├── [config directory]       # Configuration files
├── [migrations directory]   # Database schema migrations (if applicable)
├── .env.example             # Environment variable template — never touch .env
├── CONSTITUTION.md          # Non-negotiable project rules
├── AGENTS.md                # This file
└── specs/
    └── active/              # Active feature specs (requirements, design, tasks)
```

> Fill in your actual directory names. The structure above is a guide, not a rule.

---

## 4. Stack & Conventions

**Naming conventions:**

| Thing | Convention | Example |
|---|---|---|
| [e.g., Classes / Types] | [e.g., PascalCase] | [e.g., UserService] |
| [e.g., Functions / Methods] | [e.g., camelCase / snake_case] | [e.g., findUserById] |
| [e.g., Files] | [e.g., kebab-case / snake_case] | [e.g., user-service.ts] |
| [e.g., DB columns] | [e.g., snake_case] | [e.g., created_at] |
| [e.g., Constants] | [e.g., UPPER_SNAKE_CASE] | [e.g., MAX_RETRY_COUNT] |

**Code style rules:**

- [e.g., Max function/method length: 20 lines — extract if longer]
- [e.g., No null returns from public methods — use Optional or explicit error types]
- [e.g., No commented-out code — delete it or track it in a task]
- [e.g., No logging of PII, tokens, passwords, or secrets]
- [Add your own project-specific rules]

**Approved libraries / dependencies:**

| Allowed | Forbidden |
|---|---|
| [library name] — [why it's approved] | [library name] — [why it's banned] |
| [library name] — [why it's approved] | Any new dependency without human approval |

---

## 5. Git Workflow

- Branch naming: `feat/short-description`, `fix/short-description`, `chore/short-description`
- Commit format: `feat: add login page`, `fix: correct auth redirect`, `chore: update deps`
- Spec commits first: `spec: update intent for [feature]`, then `plan: ...`, then `feat: ...`
- Always branch from `main`
- Never force-push to `main` or any shared branch
- PRs must pass all tests and checks before merging

---

## 6. Boundaries — Safety Rules

### Always do
- Re-read `CONSTITUTION.md` before starting each task — follow every rule without exception
- Follow spec-first order: update requirements → update design/tasks → implement → verify
- Run the full test suite before marking any task done
- Add or update tests when changing any business logic
- Follow the existing code style of the file you're editing
- Update `AGENTS.md` if the project map or commands change

### Ask before doing
- Adding any new dependency
- Changing the database schema or migrations
- Modifying environment variable names or structure
- Changing a public API contract, public interface, or shared data structure
- Refactoring more than the files listed in the current task
- Introducing a new pattern not already used in the project

### Never do
- Read, log, commit, or expose secrets, passwords, API keys, or credentials — anywhere
- Bypass or weaken authentication or authorization logic
- Log sensitive data — passwords, tokens, emails, PII must never appear in log output
- Delete files without explicit instruction
- Push directly to `main` or any protected branch
- Change linting, formatting, or test configuration files without approval
- Skip a failing test to make a build pass

---

## 7. Testing Expectations

- Every new feature must have at least one test covering the happy path
- Every EARS clause maps to at least one scenario and one test — see `TEST_FIRST_GATE.md`
- Write tests before implementation — a passing test before the code exists is a false green
- Test names must describe behavior: `[method]_given[Condition]_[expectedResult]` or `should [behavior] when [condition]`
- Test files live in `[test directory]` mirroring the source structure
- No real external calls in unit tests — mock or stub at system boundaries
- Aim for behavior testing, not implementation testing

---

## 8. Architecture Rules

[Fill in your project-specific architecture rules. Examples:]

- [e.g., Business logic lives in the service layer only — controllers are thin]
- [e.g., Dependencies flow inward — domain has no knowledge of infrastructure]
- [e.g., Every new endpoint must have a corresponding OpenAPI annotation]
- [e.g., Database queries go through the repository layer only]

For the full rule set, see `CONSTITUTION.md`.

---

## Notes for the AI

- When in doubt, ask. A short clarifying question is always better than a wrong assumption.
- If a task touches authentication, authorization, payments, or PII — stop and confirm before proceeding.
- Prefer small, focused changes. Avoid large refactors unless explicitly asked.
- This file is the source of truth for operational context. `CONSTITUTION.md` is the source of truth for rules. If they conflict, flag it.
