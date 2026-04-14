# RUNBOOK — Spec-First Diagnostic Checklist

> A fillable checklist for one diagnostic session.
> Every step produces an artifact. No step is skipped.
> Copy this file to `example-diagnostic/` and fill it in as you go.

---

## Before You Start

**The rule:** Check the spec before you look at any code.
If no EARS clause exists for the symptom, write one now. That is Step 1.

---

## Step 1 — Pick the Spec Clause

**Fill in:**

```
Feature: _______________________________________________

Source EARS clause:

  WHEN _______________________________________________,
    the system SHALL _______________________________________________
    _______________________________________________

What triggered this session: _______________________________________________

Why this is a spec issue (not just a bug): _______________________________________________
```

**Tools needed for this step:** None. Spec only.

**Done when:** You can state exactly what "fixed" looks like before any tool is called.

**Output file:** `spec-requirement.md`

---

## Step 2 — Reproduce

**Goal:** Recreate the exact user journey described by the spec clause.
No creative exploration — follow the spec's preconditions literally.

**Fill in:**

```
Playwright session plan:

  1. navigate to: _______________________________________________
  2. action: _______________________________________________
  3. action: _______________________________________________
  4. assert: _______________________________________________

Environment:        _______________________________________________
Test data / seed:   _______________________________________________
```

**Done when:** The symptom reproduces reliably. If it doesn't, record that — it's a valid outcome.

---

## Step 3 — Record Evidence

**Goal:** Capture what the system actually did. Tag every observation to a spec clause.

**Fill in:**

```
Evidence #1
  Tool: _______________________________________________
  Spec clause: _______________________________________________
  Raw result:

    _______________________________________________

  Verdict: [ ] REQUIREMENT VIOLATED  [ ] REQUIREMENT MET  [ ] INCONCLUSIVE

Evidence #2
  Tool: _______________________________________________
  Spec clause: _______________________________________________
  Raw result:

    _______________________________________________

  Verdict: [ ] REQUIREMENT VIOLATED  [ ] REQUIREMENT MET  [ ] INCONCLUSIVE
```

**Rules:**
- Every evidence item must cite a spec clause. If it doesn't map to a clause, note it as "supporting — no direct clause."
- Screenshots count. Network logs count. Console errors count.
- Do not interpret here — just record.

**Output file:** `evidence-log.md`

---

## Step 4 — Localize

**Goal:** Map each piece of evidence to a specific file, line, and function.
No fix proposed here — only cause and proof.

**Fill in:**

```
Root Cause #1
  Evidence that points here: _______________________________________________
  Spec clause violated: _______________________________________________
  File: _______________________________________________  Line: ___
  Current code:

    _______________________________________________

  Why this causes the violation:

    _______________________________________________

Root Cause #2
  Evidence that points here: _______________________________________________
  Spec clause violated: _______________________________________________
  File: _______________________________________________  Line: ___
  Current code:

    _______________________________________________

  Why this causes the violation:

    _______________________________________________
```

**Done when:** You can point to the exact line that caused each evidence observation.

**Output file:** `root-cause.md`

---

## Step 5 — Propose Fix

**Goal:** One targeted change per root cause. Each fix must include proof steps — the exact observations that confirm the violation is resolved.

**Fill in:**

```
Fix #1
  Addresses: Root Cause #1 (file:line)
  Expected impact: _______________________________________________

  Change:
    BEFORE: _______________________________________________
    AFTER:  _______________________________________________

  Proof — Given / When / Then:
    Given  _______________________________________________
    When   _______________________________________________
    Then   _______________________________________________
      And  _______________________________________________
```

**Rules:**
- One fix per root cause.
- If a fix is deferred (out of scope, needs product decision), document the `[NEEDS_CLARIFICATION]` item explicitly.
- Do not bundle fixes. Bundled fixes make proof steps unverifiable.

**Output file:** `fix-proposal.md`

---

## Step 6 — Update the Spec

**Goal:** Close the feedback loop. If this bug reached production because a requirement was too vague, sharpen it now.

**Fill in:**

```
Spec gaps found during this session:

  Gap #1
    The original spec said: _______________________________________________
    This was too vague because: _______________________________________________
    Proposed EARS update:

      WHEN _______________________________________________,
        the system SHALL _______________________________________________
        _______________________________________________

  Gap #2 (if any)
    _______________________________________________
```

**Done when:**
- [ ] All violated requirements are now testable and precise
- [ ] All fix proof steps have been executed and passed
- [ ] No "seems better" — the spec clause passes or a sharper clause is written

**If no spec gaps:** Write "No spec gaps. Requirements were precise enough to catch this automatically." — and note what caught it (test, alert, user report).

---

## Session Closure Checklist

```
[ ] spec-requirement.md  — EARS clause documented, diagnostic goal defined
[ ] evidence-log.md      — all observations tagged to spec clauses
[ ] root-cause.md        — every cause localized to file + line
[ ] fix-proposal.md      — targeted fixes with Given/When/Then proof
[ ] spec updated         — vague clauses sharpened (or "no gaps found")
[ ] session closed with  — passing requirement OR sharper spec (never "seems better")
```
