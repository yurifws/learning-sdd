# spec-requirement.md — Dashboard LCP Regression

> **Step 1 of the Runtime Gateflow.**
> Every diagnostic session starts here — with a single, specific requirement from the spec.
> Never begin without a spec anchor.

---

## Feature

**Dashboard — Hero Section Load Performance (Core Web Vitals)**

---

## Source Requirement (EARS)

```
WHEN the user navigates to /dashboard,
  the system SHALL achieve a Largest Contentful Paint (LCP)
  below 2.5 seconds as measured by Web Vitals
  on a standard broadband connection (10Mbps+).
```

---

## What Triggered This Investigation

**Alert:** Automated Web Vitals monitoring flagged a regression — LCP climbed from 1.8s to 4.1s after the `2026-04-10` release. No user-facing feature changed visibly. The regression is consistent across runs.

**Why this is a spec issue, not just a performance complaint:**
LCP < 2.5s is a named requirement in `requirements.md §3.1`. The spec defines exactly what "good" looks like. The question is not "is it slow?" but "does it violate §3.1?" — and the alert says yes.

---

## Diagnostic Goal

Produce evidence that either:

1. **Confirms the regression** — LCP consistently exceeds 2.5s, with root cause localized to a specific file and line that changed in the `2026-04-10` release
2. **Clears the requirement** — LCP meets spec and the alert is environmental (test infrastructure, not production)

Either outcome is useful. "It might be the script tag" is not.

---

## Session Scope

| Tool | Permitted actions |
|---|---|
| Playwright | navigate, screenshot, evaluate |
| DevTools | startTrace, stopTrace, queryTrace, getNetworkLog, getLCPTimeline |
| Figma | not required |

**Environment:** `http://localhost:3000`
**Branch:** `release/2026-04-10` (regression branch) vs `main` (baseline)

---

## Definition of Done

The diagnostic session is complete when:

- [ ] LCP trace captured — exact timestamp and element identified
- [ ] Long task identified — file, function, and duration recorded
- [ ] Root cause mapped to the specific change introduced in `2026-04-10`
- [ ] Fix proposed in `fix-proposal.md` with proof steps
- [ ] Spec gap assessed — was the requirement precise enough to catch this in CI?
