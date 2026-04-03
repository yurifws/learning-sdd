# AGENTS.md — AI Onboarding Packet

> This file is written for AI agents, not humans.
> Its purpose is to give the AI all the operational context it needs to work effectively on this project.

---

## 1. Project Map

```
my-project/
├── src/
│   ├── app/          # Main application code
│   ├── components/   # Reusable UI components
│   ├── lib/          # Utility functions and helpers
│   └── types/        # TypeScript type definitions
├── tests/            # All test files (mirror src/ structure)
├── docs/             # Documentation for humans
├── public/           # Static assets
├── .env.example      # Environment variable template (never touch .env)
└── AGENTS.md         # This file
```

---

## 2. Commands

| Task              | Command                    |
|-------------------|----------------------------|
| Install deps      | `pnpm install`             |
| Start dev server  | `pnpm dev`                 |
| Run all tests     | `pnpm test`                |
| Run single test   | `pnpm test path/to/file`   |
| Lint              | `pnpm lint`                |
| Format code       | `pnpm format`              |
| Type check        | `pnpm typecheck`           |
| Build production  | `pnpm build`               |

> Always use `pnpm`. Never use `npm` or `yarn`.

---

## 3. Stack & Conventions

- **Language**: TypeScript (strict mode enabled)
- **Framework**: Next.js 14 with App Router
- **Styling**: Tailwind CSS — no inline styles, no CSS modules
- **Testing**: Vitest + React Testing Library
- **Linting**: ESLint + Prettier (config already in repo — don't change it)

### Code style rules
- Use named exports, not default exports (except for Next.js pages)
- Prefer `const` arrow functions over `function` declarations
- Keep components under 150 lines — split if larger
- No `any` types — use `unknown` and narrow it
- File names: `kebab-case.ts`, Component names: `PascalCase`

---

## 4. Git Workflow

- Branch naming: `feat/short-description`, `fix/short-description`, `chore/short-description`
- Commit format: `feat: add login page`, `fix: correct auth redirect`, `chore: update deps`
- Always branch from `main`
- Never force-push to `main` or `staging`
- PRs must pass all tests and type checks before merging

---

## 5. Boundaries (Safety Rules)

### Always do
- Run `pnpm test` and `pnpm typecheck` before considering a task done
- Add or update tests when changing business logic
- Follow the existing code style of the file you're editing
- Ask before adding a new dependency

### Ask before doing
- Adding any `npm`/`pnpm` package
- Changing the database schema or migrations
- Modifying environment variable names or structure
- Refactoring more than one file at a time
- Changing shared utility functions in `src/lib/`

### Never do
- Read, log, or commit `.env` files or secrets
- Bypass or modify authentication/authorization logic in `src/app/auth/`
- Delete files without explicit instruction
- Push directly to `main` or `staging`
- Change linting or formatting config files
- Install packages not listed in `package.json`

---

## 6. Testing Expectations

- Every new feature must have at least one unit test
- Test files live in `tests/` mirroring the `src/` structure
- Use descriptive test names: `it('should redirect unauthenticated users to login')`
- Mock external APIs — never make real HTTP calls in tests
- Aim for behavior testing, not implementation testing

---

## Notes for the AI

- When in doubt, ask. A short clarifying question is always better than a wrong assumption.
- If a task touches `src/auth/`, stop and confirm before proceeding.
- Prefer small, focused changes. Avoid large refactors unless explicitly asked.
- This file is the source of truth. If it conflicts with something in the README, follow this file and flag the discrepancy.
