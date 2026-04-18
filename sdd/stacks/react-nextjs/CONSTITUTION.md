# PROJECT CONSTITUTION — REACT / NEXT.JS FRONTEND

**Project:** [Your Project Name]
**Version:** 1.0
**Last updated:** [Date]

---

## Article 1 — Architectural principles

- **Server-first by default:** Use React Server Components (RSC) unless the component needs interactivity, browser APIs, or a hook that requires a client. Only mark `"use client"` when you actually need it — never by habit.
- **Locked stack:** [e.g. TypeScript 5.x (strict), Next.js 14 (App Router), React 18, Tailwind CSS, pnpm] — no upgrades without team sign-off.
- **Design tokens are the only source of visual truth:** colors, spacing, typography, and radii come from the token file. Never hardcode.
- **One feature = one route segment:** feature code colocates with the route under `app/[feature]/`. Shared primitives live under `components/ui/`.
- **Data fetching at the edge of the component tree:** fetch in server components or route handlers; pass the result down. Client components receive props, not data-fetching responsibilities.
- **Simplicity over cleverness:** if a component needs a comment to explain what it does, split or rename it first.
- **Immutability by default:** treat props and state as read-only; never mutate.

---

## Article 2 — Coding standards

**Naming**
- Components: `PascalCase` (`UserCard.tsx`)
- Hooks: `camelCase` starting with `use` (`useCart.ts`)
- Files: `kebab-case.ts` for utilities; `PascalCase.tsx` for components
- Types and interfaces: `PascalCase`, no `I`-prefix
- Constants: `UPPER_SNAKE_CASE` only when truly constant across the app; prefer readable names otherwise
- Route segments (App Router): `app/(group)/feature-name/page.tsx` — groups in parens, segments in kebab-case

**Project structure (App Router)**
```
src/
├── app/                    ← routes, layouts, route handlers — server-first
│   ├── (marketing)/        ← route groups
│   └── [feature]/
│       ├── page.tsx        ← server component by default
│       ├── layout.tsx
│       └── actions.ts      ← server actions
├── components/
│   ├── ui/                 ← stateless primitives (Button, Input, Card)
│   └── [feature]/          ← feature-specific components
├── lib/                    ← pure utilities, no React imports
├── hooks/                  ← custom React hooks (client only)
├── server/                 ← server-only modules (DB, external APIs, secrets)
├── styles/                 ← global CSS, token imports
└── types/                  ← shared TypeScript types
```

**TypeScript**
- `strict: true` in `tsconfig.json` — no exceptions
- No `any` — use `unknown` and narrow it
- No non-null assertions (`value!`) — handle `null`/`undefined` explicitly
- Prefer `type` over `interface` for prop types; use `interface` only when declaration merging is needed
- Component props are typed, not inferred: `export function Card(props: CardProps)`

**Styling**
- Tailwind utility classes only — no inline `style={}`, no CSS-in-JS, no CSS modules
- Every color/spacing/radius/font value comes from the token file — never raw hex, never raw `px`
- If a token is missing, stop and flag `[NEEDS_CLARIFICATION]` — do not invent one
- Use `clsx` (or `cn`) for conditional classes; never string concatenation

**Code style**
- Named exports only (except Next.js `page.tsx` / `layout.tsx` which require `default`)
- Components under 150 lines — extract subcomponents if larger
- Arrow function components: `export const Foo = (props: FooProps) => { ... }`
- Avoid default props; use required props with sensible prop-level defaults via destructuring
- No `console.log` in committed code — use a logger and gate with env

**Dependencies (pnpm)**

| Allowed | Forbidden |
|---|---|
| `next`, `react`, `react-dom` (official) | Any lib not on npm |
| `@tanstack/react-query`, `zustand` (state) | Redux unless already in use |
| `zod` (runtime validation at boundaries) | `yup`, `joi` — pick one, stay consistent |
| `@testing-library/react`, `vitest`, `playwright` | `jest` unless already in use |
| `clsx`, `tailwind-merge` | `styled-components`, `emotion`, `@mui/*` |
| `date-fns` | `moment` (deprecated) |
| [add your approved libs] | Any lib with fewer than 1000 GitHub stars |
| [add your approved libs] | Any lib unmaintained > 2 years |

---

## Article 3 — Testing rules (non-negotiable)

- **Test-first:** write the failing test before the implementation for any new feature or bug fix
- **Coverage floor:** minimum [80]% line coverage — PRs below this are rejected
- **No `.skip` or `.only` in committed code:** fix it or delete it
- **Unit tests (Vitest):** for hooks, utilities in `lib/`, and pure logic — no DOM needed
- **Component tests (Testing Library):** user-facing behavior only. Query by role/label, never by class name or test-id unless there's no other way
- **E2E tests (Playwright):** every critical user flow — login, checkout, data entry — has one end-to-end scenario
- **Required states must have tests:** loading, empty, error, and success — every async component covers all four
- **Test naming:** `describe("<Component>")` → `it("renders X when Y")`

**Test stack**
- Unit/component: Vitest + React Testing Library
- E2E: Playwright
- No Enzyme, no shallow rendering

---

## Article 4 — Accessibility (non-negotiable)

- Target: **WCAG 2.1 AA** across every shipped screen
- Every interactive element is reachable by keyboard; focus ring is always visible
- Every form input has an associated `<label>` (or `aria-label` when visually hidden)
- Every image has `alt` text (or `alt=""` for decorative)
- Color is never the only signal — status also uses text, icon, or shape
- Run `axe` checks in CI; fail the build on new violations

---

## Article 5 — The 3-tier enforcement system

### Always do (no questions asked)
- Re-read `CONSTITUTION.md` before starting each task — follow every rule without exception
- Follow spec-first order: update visual spec → update plan/tasks → implement → verify against the reference
- Commit in order: spec update first, plan/tasks second, implementation last
- Run `pnpm typecheck && pnpm lint && pnpm test` before every commit
- Use the token file for every visual value
- Implement all required states (loading, empty, error, success) listed in the visual spec
- Keep PRs small — one feature or fix per PR, max ~400 lines changed

### Ask a human first
- Adding any `pnpm` dependency
- Changing the token file or adding a new token
- Modifying shared layout or global styles (`app/layout.tsx`, global CSS)
- Adding or changing a route segment structure (renames, groupings)
- Introducing a new state-management tool or data-fetching pattern
- Any performance optimization that adds significant complexity

### Never do (hard red lines)
- Commit secrets, API keys, or `.env*` files
- Read server-only values on the client — `NEXT_PUBLIC_*` is the only public prefix
- Use `dangerouslySetInnerHTML` without an explicit sanitizer and a human-approved comment explaining why
- Use `any`, non-null assertions (`!`), or `// @ts-ignore`
- Hardcode colors, spacing, font families, or radii
- Invent components, layouts, or interactions not in the visual spec
- Skip the visual-notes step in the pixel-to-requirement pipeline
- Push directly to `main` or `staging`
- Disable or skip failing tests to make a PR pass
- Use a forbidden or unapproved dependency

---

## Article 6 — Architecture gates

Before writing any code, answer these questions. If you can't answer them clearly, stop and ask.

**Simplicity gate**
> "Is this complexity actually justified by the visual spec and requirements?"

**Server / client gate**
> "Does this component need to be a client component, or can it stay server-rendered?"

**Token gate**
> "Is every visual value I'm about to write coming from a token? If not, which token is missing?"

**State gate**
> "Does this state actually need to live in React state, or can it be derived from props or URL?"

**Data-fetching gate**
> "Am I fetching from the right place — server component, route handler, or server action? Am I pushing data-fetching into a client component unnecessarily?"

**Accessibility gate**
> "Is every interactive element reachable by keyboard, labeled, and distinguishable by something other than color alone?"

---

## Article 7 — Performance budgets

Enforced in CI via Lighthouse or WebPageTest on the staging deployment.

| Metric | Budget | Measured on |
|---|---|---|
| Largest Contentful Paint (LCP) | ≤ 2.5s | p75, mobile 4G |
| Interaction to Next Paint (INP) | ≤ 200ms | p75 |
| Cumulative Layout Shift (CLS) | ≤ 0.1 | p75 |
| First-party JS on initial route | ≤ 150 KB gzipped | per route |
| Images | `next/image` required; no raw `<img>` in `app/` | all routes |
| Fonts | `next/font` required; no `@import` of fonts in CSS | all routes |

When a budget is violated: stop feature work, fix or ask. No "we'll optimize later."

---

## Article 8 — Security

- All user input is validated at the boundary with Zod before reaching any server logic
- All server actions and route handlers check authorization before doing anything
- CSRF protection is enabled for mutating requests (default in Next.js server actions — do not disable)
- Never echo user input into HTML without escaping; the default React behavior is safe — don't circumvent it with `dangerouslySetInnerHTML`
- Session tokens live in HTTP-only cookies — never in `localStorage`
- Error messages never leak stack traces or internal IDs to the client

---

## Article 9 — Glossary

| Term | Definition |
|---|---|
| Server component | A component rendered on the server, cannot use hooks or browser APIs. The default in the App Router. |
| Client component | A component marked with `"use client"`, required for interactivity, effects, and browser APIs. |
| Server action | A function marked with `"use server"` that runs on the server and can be invoked from a client component. |
| Token | A named design value (color, spacing, radius, font) — the only legal source of visual values. |
| Route segment | A folder under `app/` that contributes a path to the URL. |
| [Term] | [What it means in this project] |

---

*This constitution applies to all contributors — human and AI alike. When in doubt, refer here first. Paired with [`VISUAL_SPEC.md`](VISUAL_SPEC.md) for the pixel-to-requirement pipeline.*
