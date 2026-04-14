# ui-contract.md — UI Contract

> **Produced by:** UI Agent
> **Consumed by:** QA Agent
> **Status:** LOCKED — QA Agent has started.

---

## 1. Components

### `<TaskDashboard />`

Top-level page component. Owns auth check, data fetching, and layout.

**Props:** none (reads auth token from context)

**States:**

| State | Value | What renders |
|---|---|---|
| `loading` | `true` | `<LoadingSpinner />` |
| `error` | non-null | `<ErrorBanner message={...} onRetry={fetch} />` |
| `tasks.length === 0` | — | `<EmptyState />` |
| default | — | `<TaskList />` + `<CreateTaskForm />` + `<Pagination />` |

---

### `<TaskList tasks={Task[]} />`

Renders the list. Each item is a `<TaskCard />`.

**Props:**
```typescript
tasks: Task[]   // Task type from api-contract.md response shape
```

---

### `<TaskCard task={Task} />`

Displays a single task. Read-only in Phase 1.

**Props:**
```typescript
task: { id: number; title: string; status: string; createdAt: string }
```

**Renders:** title, status badge, formatted `createdAt` (`DD MMM YYYY`)

---

### `<CreateTaskForm onCreated={(task: Task) => void} />`

Form with a single `title` input and a Submit button.

**Behaviour:**
- Submit disabled while request is in-flight
- On success: clears input, calls `onCreated(task)`, shows `<SuccessToast message="Task created" />` for 3 seconds
- On 400: shows inline field error below the input
- On network error: shows `<ErrorBanner />` above the form

---

### `<Pagination page={number} totalPages={number} onPageChange={(p) => void} />`

Renders "Previous" / "Next" buttons.
Previous disabled when `page === 0`. Next disabled when `page === totalPages - 1`.
Hidden when `totalPages <= 1`.

---

### `<LoadingSpinner />`

Accessible spinner: `role="status"`, `aria-label="Loading tasks"`.

---

### `<EmptyState />`

Renders: `"No tasks yet. Create your first one."`

---

### `<ErrorBanner message={string} onRetry={() => void} />`

Renders the error message and a Retry button.

---

### `<SuccessToast message={string} />`

Auto-dismisses after 3 seconds. Accessible: `role="alert"`.

---

## 2. API Client

```typescript
// src/api/taskApi.ts

export const taskApi = {
  list: (page: number, size: number): Promise<ListResponse> =>
    fetch(`/api/tasks?page=${page}&size=${size}`, { headers: authHeader() }),

  create: (title: string): Promise<Task> =>
    fetch('/api/tasks', {
      method: 'POST',
      headers: { ...authHeader(), 'Content-Type': 'application/json' },
      body: JSON.stringify({ title }),
    }),
};
```

Error handling: non-2xx responses are parsed as `{ error, message }` and re-thrown as typed errors.

---

## 3. Gate D Sign-Off

- [x] All components defined with props and state documented
- [x] Empty state, loading, and error states covered (all EARS §2.2 states)
- [x] Form behaviour matches EARS §2.3 (success toast, inline error, clears on success)
- [x] Pagination matches EARS §2.4 (Previous/Next, hidden when single page)
- [x] API client typed against api-contract.md response shapes

**Status: LOCKED. QA Agent may begin.**
