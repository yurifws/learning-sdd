# steering/style.md

> Naming conventions, formatting rules, and team preferences.
> The AI reads this before every session. These rules apply to every file it touches.

---

## Naming Conventions

### Java

| Element | Convention | Example |
|---|---|---|
| Class | PascalCase | `LinkService`, `ExpiredLinkException` |
| Method | camelCase, verb-first | `findByToken()`, `markAsExpired()` |
| Variable | camelCase | `shortLink`, `expiresAt` |
| Constant | UPPER_SNAKE_CASE | `MAX_EXPIRY_DAYS` |
| Package | lowercase, no underscores | `com.example.link` |
| Test class | `{Subject}Test` | `LinkServiceTest` |
| Integration test class | `{Subject}IT` | `LinkControllerIT` |

### Database

| Element | Convention | Example |
|---|---|---|
| Table | snake_case, plural | `short_links`, `click_events` |
| Column | snake_case | `expires_at`, `created_by` |
| FK column | `{table_singular}_id` | `short_link_id` |
| Index | `idx_{table}_{column}` | `idx_short_links_token` |
| Migration file | `V{n}__{Description}.sql` | `V3__Add_expiry_column.sql` |

### REST Endpoints

| Rule | Example |
|---|---|
| Plural nouns, lowercase | `/api/v1/links` |
| No verbs in path | `/api/v1/links/{id}/deactivate` not `/deactivateLink` |
| Version prefix always | `/api/v1/...` |

---

## Code Style

- Max line length: **120 characters**
- Indentation: **4 spaces** (no tabs)
- One blank line between methods
- No trailing whitespace
- `@Override` always present when applicable
- Javadoc only on public interfaces and their methods — not on implementations

---

## Test Style

- One `@Test` per behavior — not one test per method
- Test method name: `should{Result}_when{Condition}`
  - Example: `shouldReturn410_whenLinkIsExpired()`
- Use `@DisplayName` for human-readable output
- `// given / // when / // then` comments inside each test — always
- No production code in test setup that belongs in Flyway migrations

---

## Git Conventions

| Type | Commit prefix | When to use |
|---|---|---|
| Spec | `spec: ` | requirements.md, design.md, tasks.md |
| Feature | `feat: ` | production code |
| Test | `test: ` | test files only |
| Fix | `fix: ` | bug correction |
| Refactor | `refactor: ` | no behavior change |
| Chore | `chore: ` | build, deps, config |

> Rule: a spec commit and a code commit are always separate. Never mix them.

---

## What NOT to Do

- Do not add comments that restate what the code already says
- Do not introduce abstractions for a single use case
- Do not rename symbols that are not in scope for the current task
- Do not add `TODO` or `FIXME` without a linked issue number
