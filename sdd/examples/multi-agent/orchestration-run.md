# orchestration-run.md — Full-Stack Orchestration Walkthrough

> **Feature:** Task Dashboard (FEAT-002)
> **Agents:** DB · API · UI · QA
> **What this file shows:** The exact prompt each agent receives, what it reports back,
> what the human checks at each gate, and how a [NEEDS_CLARIFICATION] gets resolved.

---

## How to Read This File

- `[YOU → ORCHESTRATOR]` — the prompt you send to kick off the session
- `[ORCHESTRATOR]` — what the orchestrator decides and communicates
- `[ORCHESTRATOR → AGENT]` — the exact prompt sent to a specialist
- `[AGENT → ORCHESTRATOR]` — what the specialist reports when done
- `[GATE: HUMAN]` — what you review and approve before the next agent starts
- `[NEEDS_CLARIFICATION]` — a blocker example and how it resolves

---

## Step 0 — Kickoff

### [YOU → ORCHESTRATOR]

```
You are the orchestrator for FEAT-002 — Task Dashboard.

Read these files before doing anything:
  1. requirements.md     — EARS rules for list, create, and pagination
  2. db-contract.md      — filled in after the DB Agent finishes (check status)
  3. api-contract.md     — filled in after the API Agent finishes (check status)
  4. ui-contract.md      — filled in after the UI Agent finishes (check status)

Your job:
  - Delegate work to four specialist agents in order: DB → API → UI → QA
  - Give each agent ONLY the contracts from upstream layers — nothing else
  - Do NOT write any code yourself
  - After each agent reports done, run their verify command and confirm it passes
  - Wait for my Gate approval before starting the next agent
  - If any agent flags [NEEDS_CLARIFICATION], stop all work and report to me immediately

Start by summarising the feature and the agent sequence. Wait for my confirmation.
```

---

### [ORCHESTRATOR]

```
Summary:

FEATURE: Task Dashboard (FEAT-002)
  A web page where authenticated users can see their tasks (paginated, 20/page)
  and create new ones via a form.
  Out of scope: edit, delete, filter (Phase 2).

AGENT SEQUENCE (sequential — each depends on the layer below):
  1. DB Agent    → schema + data access layer → produces db-contract.md (Gate B)
  2. API Agent   → REST endpoints             → reads db-contract.md → produces api-contract.md (Gate C)
  3. UI Agent    → React components           → reads api-contract.md → produces ui-contract.md (Gate D)
  4. QA Agent    → all tests                 → reads all three contracts (Gate E)

Agents 3 (UI) and 4 (QA) could run in parallel once api-contract.md is LOCKED —
but I will run them sequentially unless you instruct otherwise.

Ready to start DB Agent. Please confirm.
```

### [GATE: HUMAN — Pre-flight]

```
✅ Confirmed. Start with the DB Agent. Run UI and QA in parallel after Gate C.
```

---

## Step 1 — DB Agent

### [ORCHESTRATOR → DB Agent]

```
You are the DB Agent for FEAT-002 (Task Dashboard).

Read this file before starting:
  - requirements.md   — understand what data the feature needs

Your task:

  1. Write the Flyway migration:
       migrations/001_create_tasks.sql
     Table: tasks (id SERIAL PK, user_id INTEGER NOT NULL FK→users.id,
                   title VARCHAR(255) NOT NULL, status VARCHAR(20) NOT NULL DEFAULT 'TODO',
                   created_at TIMESTAMPTZ NOT NULL DEFAULT NOW())
     Index: idx_tasks_user_created on (user_id, created_at DESC)

  2. Implement the data access layer:
       src/db/taskRepository.ts
     Export a TaskRepository interface and a PostgresTaskRepository class.
     Methods:
       create(userId: number, title: string): Promise<Task>
       findAllByUser(userId: number, page: number, size: number):
         Promise<{ tasks: Task[]; totalCount: number }>
     findAllByUser must exclude DELETED tasks and order by created_at DESC.

  3. Fill in db-contract.md:
     Complete the Gate B sign-off section. Set Status to LOCKED.

Rules:
  - Touch ONLY the two files above + db-contract.md
  - No business logic — only SQL and type mapping
  - Use parameterised queries — no string interpolation in SQL
  - If you need anything not in requirements.md, STOP and flag [NEEDS_CLARIFICATION]

When done, run:
  npm test -- --testPathPattern=taskRepository
Report the result.
```

---

### [NEEDS_CLARIFICATION — example]

```
[DB Agent → ORCHESTRATOR]

[NEEDS_CLARIFICATION]

requirements.md §2.3 says POST /api/tasks creates a task, but it does not specify
the initial value for the `status` column.

I see two options:
  A) Always insert with status = 'TODO' (hardcoded in the repository)
  B) Accept status as a parameter to create()

I have not written any code yet. Please clarify.
```

### [GATE: HUMAN — Resolving the blocker]

```
Option A. Status is always TODO on creation. The repository hardcodes it.
The API layer never passes a status to create().
```

```
[ORCHESTRATOR → DB Agent]

Clarification: status is always TODO on creation. Hardcode it in the repository.
create() signature stays: create(userId: number, title: string): Promise<Task>
Continue from where you stopped.
```

---

### [DB Agent → ORCHESTRATOR] — after clarification

```
DB Agent done.

Files created:
  migrations/001_create_tasks.sql   — tasks table + index, idempotent (IF NOT EXISTS)
  src/db/taskRepository.ts          — TaskRepository interface + PostgresTaskRepository
    create():         INSERT with status='TODO', returns saved Task
    findAllByUser():  SELECT WHERE user_id=? AND status != 'DELETED'
                      ORDER BY created_at DESC, LIMIT/OFFSET for pagination

db-contract.md: Gate B sign-off completed. Status set to LOCKED.

Verify:
  npm test -- --testPathPattern=taskRepository
  ✓ creates a task and returns it with id and createdAt
  ✓ findAllByUser returns tasks in created_at DESC order
  ✓ findAllByUser excludes DELETED tasks
  ✓ findAllByUser returns correct totalCount for pagination
  4 tests passing
```

### [GATE B: HUMAN]

Open `db-contract.md`. Check:

- [ ] Table schema matches `requirements.md` (title, status, timestamps, user FK)
- [ ] `TaskRepository.create()` does not accept a status param — always TODO
- [ ] `findAllByUser` excludes DELETED tasks (confirmed in test output)
- [ ] Migration is idempotent (`IF NOT EXISTS`)
- [ ] 4 repository tests passing

```
✅ Gate B passed. db-contract.md is LOCKED. Start the API Agent.
```

---

## Step 2 — API Agent

### [ORCHESTRATOR → API Agent]

```
You are the API Agent for FEAT-002 (Task Dashboard).

Read these files before starting:
  1. requirements.md   — EARS rules for the two endpoints
  2. db-contract.md    — the only data layer interface you may use
                         DO NOT read taskRepository.ts directly

Your context from db-contract.md:
  TaskRepository methods: create(userId, title), findAllByUser(userId, page, size)
  Task type: { id, userId, title, status, createdAt }
  findAllByUser returns: { tasks: Task[], totalCount: number }

Your task:

  1. Create the task router:
       src/routes/taskRoutes.ts
     GET  /api/tasks   — list tasks (query params: page=0, size=20, max size=100)
     POST /api/tasks   — create task (body: { title })

  2. Create the task controller:
       src/controllers/taskController.ts
     list():   extract userId from JWT → call taskRepository.findAllByUser()
               → return { tasks, page, size, totalCount, totalPages }
     create(): validate title (required, max 255) → call taskRepository.create()
               → return created task with HTTP 201

  3. Fill in api-contract.md:
     Complete the Gate C sign-off. Set Status to LOCKED.

Rules:
  - Touch ONLY the three files above + api-contract.md
  - Import TaskRepository from db-contract — never import PostgresTaskRepository directly
  - Inject TaskRepository via constructor (do not instantiate inside controller)
  - Return structured JSON errors — no stack traces, no Express default handler
  - If size > 100 return 400. If title is blank or > 255 chars return 400.
  - If you need anything not covered, STOP and flag [NEEDS_CLARIFICATION]

When done, run:
  npm test -- --testPathPattern=taskController
Report the result.
```

---

### [API Agent → ORCHESTRATOR]

```
API Agent done.

Files created:
  src/routes/taskRoutes.ts       — GET /api/tasks, POST /api/tasks, JWT middleware wired
  src/controllers/taskController.ts — list() and create() with validation

Key decisions:
  JWT: extracted userId from token.sub (parsed as integer)
  list(): size > 100 → 400 VALIDATION_ERROR
  create(): blank title → 400, title.length > 255 → 400
  totalPages: Math.ceil(totalCount / size)
  All errors: { error: ERROR_CODE, message: optional string }

TaskRepository injected via constructor — no direct PostgresTaskRepository import.

api-contract.md: Gate C sign-off completed. Status set to LOCKED.

Verify:
  npm test -- --testPathPattern=taskController
  ✓ GET /api/tasks returns 200 with paginated task list
  ✓ GET /api/tasks with size=101 returns 400 VALIDATION_ERROR
  ✓ GET /api/tasks without JWT returns 401
  ✓ POST /api/tasks returns 201 with created task
  ✓ POST /api/tasks with blank title returns 400 VALIDATION_ERROR
  ✓ POST /api/tasks with title >255 chars returns 400 VALIDATION_ERROR
  ✓ POST /api/tasks without JWT returns 401
  7 tests passing
```

### [GATE C: HUMAN]

Open `api-contract.md`. Check:

- [ ] Both endpoints match `requirements.md` (list paginated, create with validation)
- [ ] `totalPages` is calculated and returned
- [ ] Error codes match the contract (`VALIDATION_ERROR`, `UNAUTHORIZED`)
- [ ] No stack traces in any error response
- [ ] 7 controller tests passing

```
✅ Gate C passed. api-contract.md is LOCKED.
Start UI Agent and QA Agent in parallel.
```

---

## Step 3 — UI Agent and QA Agent (parallel)

> api-contract.md is now LOCKED. Both agents can start simultaneously — they read
> the same contract and touch completely different files.

---

### [ORCHESTRATOR → UI Agent]

```
You are the UI Agent for FEAT-002 (Task Dashboard).

Read these files before starting:
  1. requirements.md   — UX behaviour (loading state, empty state, error state,
                          success toast, pagination)
  2. api-contract.md   — the only API you may call (DO NOT read any backend files)

Your task:

  Create these React/TypeScript components:
    src/components/TaskDashboard.tsx   — top-level page, owns fetch + state
    src/components/TaskList.tsx        — renders list of TaskCard items
    src/components/TaskCard.tsx        — single task display (title, status, date)
    src/components/CreateTaskForm.tsx  — form with title input + submit
    src/components/Pagination.tsx      — Previous/Next, hidden when totalPages <= 1
    src/components/LoadingSpinner.tsx  — role="status", aria-label="Loading tasks"
    src/components/EmptyState.tsx      — "No tasks yet. Create your first one."
    src/components/ErrorBanner.tsx     — message + Retry button
    src/components/SuccessToast.tsx    — auto-dismiss after 3 seconds, role="alert"
    src/api/taskApi.ts                 — typed fetch calls to api-contract.md endpoints

  Then fill in ui-contract.md:
    Complete the Gate D sign-off. Set Status to LOCKED.

Rules:
  - Touch ONLY the files listed above + ui-contract.md
  - Use api-contract.md response shapes for TypeScript types — do not invent new shapes
  - All API calls go through src/api/taskApi.ts — no inline fetch in components
  - Non-2xx responses must be parsed as { error, message } and thrown as typed errors
  - CreateTaskForm: disable submit while in-flight, clear on success, show toast 3s
  - Pagination: hide when totalPages <= 1, disable Prev at page 0, Next at last page
  - If you need anything not in api-contract.md or requirements.md, STOP and flag [NEEDS_CLARIFICATION]

When done, run:
  npm test -- --testPathPattern=components
Report the result.
```

---

### [ORCHESTRATOR → QA Agent]

```
You are the QA Agent for FEAT-002 (Task Dashboard).

Read these files before starting:
  1. requirements.md   — all EARS rules you must verify
  2. db-contract.md    — the data layer contract
  3. api-contract.md   — the API contract
  4. ui-contract.md    — the UI component contract (may still be in progress — read it when ready)

Your task:

  Write these test files:

  Unit tests (Jest + Testing Library):
    src/__tests__/unit/taskRepository.test.ts  — already written by DB Agent (do not duplicate)
    src/__tests__/unit/taskController.test.ts  — already written by API Agent (do not duplicate)
    src/__tests__/unit/TaskDashboard.test.tsx  — loading state, empty state, error + retry
    src/__tests__/unit/CreateTaskForm.test.tsx — submit, validation errors, success toast
    src/__tests__/unit/Pagination.test.tsx     — hidden, prev/next disabled states

  Integration tests (Jest + supertest):
    src/__tests__/integration/tasks.test.ts
      GET  /api/tasks — happy path, empty list, pagination, size>100 error, no auth
      POST /api/tasks — happy path, blank title, title>255, no auth

  E2E tests (Cypress):
    cypress/e2e/taskDashboard.cy.ts
      - Load dashboard → see task list
      - Create task → appears at top of list, toast shows and disappears
      - Create with blank title → inline error, no request sent
      - Load next page → URL updates, new tasks shown
      - API error → error banner with Retry button

Rules:
  - Touch ONLY the test files listed above
  - Do NOT modify any source files — only test files
  - Tests must use api-contract.md types — do not invent response shapes
  - If ui-contract.md is not LOCKED yet, write UI tests last and wait for it
  - If any test reveals a bug in a contract (wrong field name, wrong status code),
    STOP and flag [NEEDS_CLARIFICATION] with the exact discrepancy

When done, run:
  npm test && npx cypress run --headless
Report the result.
```

---

### [UI Agent → ORCHESTRATOR]

```
UI Agent done.

Files created:
  src/components/TaskDashboard.tsx  — fetches on mount, owns page state, handles
                                      loading/error/empty/success states
  src/components/TaskList.tsx       — renders TaskCard per task
  src/components/TaskCard.tsx       — title, status badge, createdAt (DD MMM YYYY)
  src/components/CreateTaskForm.tsx — controlled input, submit disabled in-flight,
                                      clears and shows toast on success, inline error on 400
  src/components/Pagination.tsx     — hidden when totalPages <= 1
  src/components/LoadingSpinner.tsx — role="status", aria-label="Loading tasks"
  src/components/EmptyState.tsx     — "No tasks yet. Create your first one."
  src/components/ErrorBanner.tsx    — message + Retry button
  src/components/SuccessToast.tsx   — auto-dismiss 3s, role="alert"
  src/api/taskApi.ts                — list() and create(), typed against api-contract.md
                                      non-2xx parsed as { error, message } and thrown

ui-contract.md: Gate D sign-off completed. Status set to LOCKED.

Verify:
  npm test -- --testPathPattern=components
  ✓ TaskDashboard renders loading spinner initially
  ✓ TaskDashboard renders empty state when tasks=[]
  ✓ TaskDashboard renders error banner on API failure + Retry triggers refetch
  ✓ CreateTaskForm disables submit while in-flight
  ✓ CreateTaskForm shows success toast and clears input on success
  ✓ CreateTaskForm shows inline error on 400
  ✓ Pagination hidden when totalPages=1
  ✓ Pagination disables Prev on page 0 and Next on last page
  8 tests passing
```

### [GATE D: HUMAN]

Open `ui-contract.md`. Check:

- [ ] All EARS §2.2 states handled: loading, empty, error + retry
- [ ] EARS §2.3 behaviours: toast 3s, form clears, inline error on 400
- [ ] EARS §2.4 pagination: hidden when 1 page, Prev/Next disabled correctly
- [ ] `taskApi.ts` typed against `api-contract.md` — no invented shapes
- [ ] 8 component tests passing

```
✅ Gate D passed. ui-contract.md is LOCKED.
```

---

### [QA Agent → ORCHESTRATOR]

```
QA Agent done.

Files created:
  src/__tests__/unit/TaskDashboard.test.tsx   — 6 unit tests
  src/__tests__/unit/CreateTaskForm.test.tsx  — 5 unit tests
  src/__tests__/unit/Pagination.test.tsx      — 3 unit tests
  src/__tests__/integration/tasks.test.ts    — 9 integration tests (supertest)
  cypress/e2e/taskDashboard.cy.ts            — 5 E2E tests

One [NEEDS_CLARIFICATION] found and resolved mid-run:
  api-contract.md shows totalPages in the GET /api/tasks response but
  ui-contract.md's Pagination reads totalPages directly from the API response.
  Confirmed consistent — no discrepancy.

Verify:
  npm test
  ✓ 14 unit tests (TaskDashboard + CreateTaskForm + Pagination)
  ✓  9 integration tests (GET list + POST create, all error cases)
  23 tests passing, 0 failing

  npx cypress run --headless
  ✓ Load dashboard → task list visible
  ✓ Create task → appears at top, toast shown and dismissed
  ✓ Blank title → inline error, no POST request fired
  ✓ Load next page → page 2 tasks shown
  ✓ API error → error banner + Retry triggers refetch
  5 E2E tests passing
```

### [GATE E: HUMAN]

Run the full suite:

```bash
npm test && npx cypress run --headless
```

Check:
- [ ] 23 unit + integration tests passing, 0 failing
- [ ] 5 E2E tests passing
- [ ] No test invented a response shape not in `api-contract.md`
- [ ] EARS requirements are covered: list, create, pagination, all error states

```
✅ Gate E passed. FEAT-002 complete.
```

---

## Final Orchestrator Report

```
[ORCHESTRATOR]

FEAT-002 — Task Dashboard: COMPLETE

Agents:
  DB Agent   ✅  schema + repository — Gate B passed
  API Agent  ✅  routes + controller — Gate C passed
  UI Agent   ✅  components + API client — Gate D passed
  QA Agent   ✅  unit + integration + E2E — Gate E passed

Contracts produced:
  db-contract.md   → LOCKED (consumed by API Agent)
  api-contract.md  → LOCKED (consumed by UI Agent + QA Agent)
  ui-contract.md   → LOCKED (consumed by QA Agent)

Clarifications resolved: 1
  DB Agent: initial task status hardcoded to TODO in create()

Tests:
  23 unit + integration tests passing
  5 E2E tests passing

Ready for Phase 2 (edit, delete, filter) when you are.
```

---

## Key Takeaways

| What happened | Why it matters |
|---|---|
| UI and QA agents ran in parallel after Gate C | api-contract.md was LOCKED — both could safely build against the same stable interface |
| QA Agent read all three contracts, not source code | Tests are written against the agreed interface, not the implementation — they will catch drift |
| DB Agent was blocked by a [NEEDS_CLARIFICATION] | One unanswered question (initial status) stopped immediately — before any code was written on a wrong assumption |
| Each agent received only its upstream contracts | The UI Agent never saw the database schema; it cannot accidentally depend on a DB detail |
| Contracts have a status field (LOCKED / IN PROGRESS) | Downstream agents can safely check whether to start or wait |
