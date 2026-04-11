# React + Next.js — SDD Stack Guide

> Start here if you're building a frontend with React or Next.js.
> This folder contains SDD applied to frontend — visual spec methodology, design tokens, and AI onboarding.

---

## What's in This Folder

| File | What it does |
|---|---|
| [`VISUAL_SPEC.md`](VISUAL_SPEC.md) | Visual spec template — tiers, pixel-to-requirement pipeline, design token rules, required states |
| [`AGENTS.md`](AGENTS.md) | AI onboarding file — visual spec protocol, tier rules, token enforcement, safety boundaries |

---

## Reading Order

1. Read [`VISUAL_SPEC.md`](VISUAL_SPEC.md) first — understand how design maps to requirements
2. Read [`AGENTS.md`](AGENTS.md) — this is what you paste into your AI session at the start of every task

---

## Stack

| Layer | Technology |
|---|---|
| Language | TypeScript |
| Framework | React 18 + Next.js 14 (App Router) |
| Styling | Tailwind CSS + design tokens |
| State | Zustand or React Query (per feature) |
| Testing | Vitest + Testing Library + Playwright |
| Design | Figma (source of truth) |

---

## The Visual Spec Approach

Frontend SDD adds one extra step before EARS requirements: the **visual spec**.

```
Design artifact (Figma)
     ↓
Visual Spec (tiers A/B/C)    ← translate pixels to requirements
     ↓
EARS requirements             ← "SHALL render with primary-action token"
     ↓
Tasks + implementation
```

### Reference Tiers

| Tier | What it is | When to use |
|---|---|---|
| **A** | Screenshot or Figma export | Exact match required — pixel-level fidelity |
| **B** | Annotated wireframe | Layout and structure required, minor style flexibility |
| **C** | ASCII mockup or description | Rough layout, full design freedom |

---

## How to Use With the SDD Workflow

1. Copy `VISUAL_SPEC.md` into your project as a template — fill in one per feature
2. Copy `AGENTS.md` into your project root — this becomes your AI's onboarding file
3. Follow the SDD workflow from [`sdd/README.md`](../../README.md):
   - Run the Clarification Gate
   - Write the visual spec (tier A/B/C)
   - Translate to EARS requirements
   - Design component structure
   - Break into atomic tasks
   - Feed the AI one task at a time
