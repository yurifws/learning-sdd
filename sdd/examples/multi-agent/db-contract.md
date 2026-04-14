# db-contract.md — Database Contract

> **Produced by:** DB Agent
> **Consumed by:** API Agent
> **Status:** LOCKED — API Agent has started.

---

## 1. Schema

### Table: `tasks`

Migration: `migrations/001_create_tasks.sql`

| Column | Type | Constraints |
|---|---|---|
| `id` | `SERIAL` | `PRIMARY KEY` |
| `user_id` | `INTEGER` | `NOT NULL`, FK → `users(id)` |
| `title` | `VARCHAR(255)` | `NOT NULL` |
| `status` | `VARCHAR(20)` | `NOT NULL`, `DEFAULT 'TODO'` |
| `created_at` | `TIMESTAMPTZ` | `NOT NULL`, `DEFAULT NOW()` |

Index: `idx_tasks_user_created` on `(user_id, created_at DESC)`

---

## 2. Data Access Interface

The API Agent imports this module. It never touches the migration file or the raw DB driver.

```typescript
// src/db/taskRepository.ts

export interface Task {
  id: number;
  userId: number;
  title: string;
  status: 'TODO' | 'IN_PROGRESS' | 'DONE' | 'DELETED';
  createdAt: Date;
}

export interface TaskRepository {
  /**
   * Persists a new task. Returns the saved task with id and createdAt populated.
   */
  create(userId: number, title: string): Promise<Task>;

  /**
   * Returns a page of non-DELETED tasks for the given user, ordered by createdAt DESC.
   * page is zero-indexed. size must be between 1 and 100.
   */
  findAllByUser(userId: number, page: number, size: number): Promise<{
    tasks: Task[];
    totalCount: number;
  }>;
}
```

---

## 3. Gate B Sign-Off

- [x] Schema matches requirements (title, status, timestamps, user FK)
- [x] Repository interface covers all API operations (create, paginated list)
- [x] No business logic in the data layer — only SQL and mapping
- [x] Migration file is idempotent (`CREATE TABLE IF NOT EXISTS`)

**Status: LOCKED. API Agent may begin.**
