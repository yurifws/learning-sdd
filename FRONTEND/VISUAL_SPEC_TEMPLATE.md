# Visual Spec — `[Feature / Screen Name]`

> **Tier:** A — Screenshot (ground truth) | B — Wireframe (structure) | C — Hi-fi mock (pixel-perfect)
> **Feature ID:** FEAT-XXX
> **Date:** YYYY-MM-DD | **Author:** [Name]

---

## 1. Visual Reference

> Attach or link the reference image. State the tier so the agent knows the rules.

| Asset | Tier | Notes |
|---|---|---|
| `screens/[screen].png` | A / B / C | [e.g., "match layout exactly" or "structure only, tokens apply"] |

---

## 2. Screen → Feature Map

| Screen / Section | Feature | Notes |
|---|---|---|
| Header | Navigation + logo | Logo must use `token.brand.logo` |
| [Section] | [Feature] | |

---

## 3. Non-Negotiables

> These values must never be hardcoded — always use design tokens.

| Element | Token | Value (reference only) |
|---|---|---|
| Primary color | `token.color.primary` | |
| Background | `token.color.surface` | |
| Font family | `token.typography.base` | |
| Spacing unit | `token.spacing.md` | |
| Border radius | `token.radius.md` | |

---

## 4. Required States

> Every state must be implemented. No state is optional unless marked.

- [ ] Default / idle
- [ ] Loading (skeleton or spinner)
- [ ] Empty (no data)
- [ ] Error (message + retry)
- [ ] Success / confirmation
- [ ] [Feature-specific state, e.g., "Selected", "Disabled"]

---

## 5. Layout (ASCII Mockup)

> Use this to lock structure before any visual work. Fast, zero-ambiguity.

```
┌─────────────────────────────────────┐
│  [Header]                [Nav items] │
├─────────────────────────────────────┤
│  [Page Title]                        │
│  ┌───────────┐  ┌───────────────┐   │
│  │  Card A   │  │    Card B     │   │
│  └───────────┘  └───────────────┘   │
└─────────────────────────────────────┘
```

---

## 6. Pixel-to-Requirement Pipeline

> Follow these steps in order. Do not write code before step 3 is approved.

**Step 1 — Agent reads the reference image**

**Step 2 — Agent writes visual notes**
The agent must describe in plain text:
- Layout structure (grid, flex, stack)
- Component list (what elements are visible)
- Spacing and alignment observations
- Color and typography observations
- Any states visible in the reference

**Step 3 — Human reviews visual notes**
- [ ] Layout matches intent
- [ ] All components identified
- [ ] No invented elements
- [ ] Token names used, not hex values
- [ ] All required states listed

> Only after this checklist is signed off may the agent write any code.

---

## 7. Visual Parity Checklist

> Run this after implementation before opening a PR.

- [ ] UI matches the reference image (or ASCII mockup if Tier B)
- [ ] All required states are implemented and testable
- [ ] No hardcoded colors, spacing, or font values — tokens only
- [ ] Responsive behavior matches spec
- [ ] No invented components or layouts not in the reference

---

## 8. Out of Scope

- [ ] ...

---

## 9. Clarification Gate

> See `CLARIFICATION_GATE.md`. Resolve all items before step 3 above.

- [ ] `[NEEDS_CLARIFICATION]` ...
