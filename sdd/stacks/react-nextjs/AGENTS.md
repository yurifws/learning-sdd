# AGENTS.md — AI Onboarding Packet
# TypeScript / React / Next.js — Visual Spec Workflow

> Read these files in order before writing or modifying any UI code:
> 1. `CLARIFICATION_GATE.md` — flag and resolve all `[NEEDS_CLARIFICATION]` items before writing any spec
> 2. `LIVING_SPEC.md` — spec-first discipline: update spec → update plan → implement → verify
> 3. This file — visual spec protocol, tokens, safety rules

---

## 1. Commands

| Task | Command |
|---|---|
| Install deps | `pnpm install` |
| Start dev server | `pnpm dev` |
| Run all tests | `pnpm test` |
| Run single test | `pnpm test path/to/file` |
| Lint | `pnpm lint` |
| Type check | `pnpm typecheck` |
| Build | `pnpm build` |

> Always use `pnpm`. Never use `npm` or `yarn`.

---

## 2. Stack & Conventions

- **Language:** TypeScript (strict mode)
- **Framework:** Next.js (App Router)
- **Styling:** Tailwind CSS — no inline styles, no CSS modules
- **Testing:** Vitest + React Testing Library
- **Design tokens:** [token file path] — never hardcode colors, spacing, or fonts

### Code style
- Named exports only (except Next.js pages)
- `const` arrow functions over `function` declarations
- Components under 150 lines — split if larger
- No `any` types — use `unknown` and narrow it
- File names: `kebab-case.ts` · Component names: `PascalCase`

---

## 3. Visual Spec Protocol

Before writing any UI code, follow the pixel-to-requirement pipeline:

1. **Read** the visual reference in `VISUAL_SPEC.md` — note the tier (A/B/C)
2. **Write visual notes** — describe in plain text what you see:
   - Layout structure (grid, flex, stack direction)
   - Every component visible in the reference
   - Spacing and alignment observations
   - Colors and typography (use token names, not hex values)
   - States visible in the reference
3. **Wait for human approval** of your visual notes before writing any code

> Never skip step 2. Never guess what's in the image — describe only what you can see.

### Tier rules

| Tier | What it means | Your job |
|---|---|---|
| A — Screenshot | Ground truth | Match exactly |
| B — Wireframe | Structure only | Match layout; apply tokens for all visual values |
| C — Hi-fi mock | Pixel-perfect | Match every detail including spacing and typography |

---

## 4. Design Tokens

- All colors, spacing, and typography values come from the token file — never hardcode
- If a value isn't in the token file, flag it with `[NEEDS_CLARIFICATION]` and stop
- Token names describe purpose, not appearance: `token.color.primary` not `token.color.blue`

---

## 5. Git Workflow

- Branch: `feat/FEAT-XXX-short-name`, `fix/short-description`, `chore/short-description`
- Commits: `feat: add login screen`, `fix: correct button alignment`
- Never force-push to `main` or `staging`
- PRs must pass all tests and type checks

---

## 6. Boundaries — Safety Rules

### Always do
- Follow spec-first order: update visual spec → update tasks → implement → verify against reference
- If anything is ambiguous, flag it as `[NEEDS_CLARIFICATION]` and stop — never guess
- Run through the pixel-to-requirement pipeline before writing any UI code
- Use design tokens for every visual value — no hardcoded hex, px, or font names
- Run `pnpm test` and `pnpm typecheck` before marking a task done
- Implement all required states from the visual spec (loading, empty, error, success)

### Ask before doing
- Adding any `pnpm` package
- Changing the token file or design system values
- Modifying shared layout components or global styles
- Refactoring more than one component at a time

### Never do
- Invent components, layouts, or interactions not in the visual spec
- Hardcode colors, spacing, or font values
- Skip the visual notes step — write them every time
- Read, log, or commit `.env` files or secrets
- Push directly to `main` or `staging`
