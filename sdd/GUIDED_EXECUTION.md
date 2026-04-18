# Guided Execution

> The AI is a **guided implementer**, not a free agent.  
> Every change must trace back to a task. Every task must trace back to the plan.  
> Full living spec discipline → `LIVING_SPEC.md`

---

## 1. Incremental Disclosure Rules

> Only show the AI what it needs for the current task — nothing else.

- Feed **one task at a time** from `tasks.md`
- Provide **only the files listed** in that task
- Do NOT share the full codebase unless explicitly required
- Use specialist agents for focused work:

| Agent Role | Responsible For |
|------------|----------------|
| 🗄️ DB Agent | Schema, migrations, repositories |
| 🔒 Security Agent | Auth, validation, sensitive data |
| 🧪 QA Agent | Tests, edge cases, negative scenarios |
| 🔧 Core Agent | Business logic, services, controllers |

---

## 2. Executable Acceptance Criteria

> Every requirement must have a **verifiable artifact** — no "looks right to me."

**Format (EARS + validation):**  
`When [trigger], the system SHALL [behavior].`  
`Verified by: [test name / command / manual checklist]`

### ✅ Feature Criteria

| # | When... | The system SHALL... | Verified by |
|---|---------|---------------------|-------------|
| AC-01 | [trigger] | [behavior] | `[YOUR_TEST_COMMAND] [TestFile]` |
| AC-02 | [trigger] | [behavior] | `[YOUR_HTTP_CALL]` returns HTTP 4xx |
| AC-03 | [trigger] | [behavior] | Manual checklist item #[N] |

### 🔴 Negative / Security Criteria

| # | To prevent... | The system SHALL... | Verified by |
|---|--------------|---------------------|-------------|
| SEC-01 | User enumeration | Return identical response for existing and non-existing emails | `[YOUR_TEST_COMMAND] [TestFile]#[testMethod]` |
| SEC-02 | [attack/risk] | [safe behavior] | |

---

## 3. Micro PR Checklist

> Each task = one PR. If reviewing hurts your brain, the task was too big.

**Before opening the PR:**
- [ ] PR touches only the files listed in the task
- [ ] PR has one and only one objective
- [ ] All acceptance criteria for this task are verified
- [ ] No unrequested refactors included
- [ ] No new libraries added that aren't in the plan
- [ ] No edge cases "fixed" that weren't specified

**PR title format:**
- Spec PR: `[SPEC] FEAT-XXX Short description`
- Impl PR: `[IMPL] FEAT-XXX Short description`
- Single-task PR: `[T-XX] Short description of what was done`

**PR description must include:**
- Task ID or Feature ID reference (e.g., `Closes T-07` or `FEAT-XXX`)
- Verify command output or test results
- Link back to SDD / Constitution section

---

## 4. Commit Convention

> See `LIVING_SPEC.md` for full rationale.

Structure every feature's commits in this order:

| Commit | Contains | Message format |
|---|---|---|
| 1st | Spec update only | `spec: update intent for [feature]` |
| 2nd | Plan + tasks update | `plan: update architecture for [feature]` |
| 3+ | Implementation | `feat: [T-XX] short description` |

This enforces the spec-first discipline right in your Git history.

---

## 5. Drift Detection — Red Flags 🚩

> If any of these appear in a PR, it is a **hard stop**. Update the plan first.

| Red Flag | What It Means | Action |
|----------|--------------|--------|
| New library not in Constitution | AI went off-plan | Reject PR, update plan if needed |
| Files touched not listed in task | Scope creep | Reject PR, split into smaller tasks |
| Edge case fixed that wasn't specified | Unauthorized change | Reject PR, create a new task for it |
| Refactor outside task scope | Classic drift | Reject PR, separate refactor task |
| Behavior change with no task reference | Untraceable change | Reject PR immediately |

---

## 6. Constitution Reference

> Reference your `CONSTITUTION.md` directly — do not duplicate its content here, as copies drift.
> Instead, ensure the AI always reads `CONSTITUTION.md` before starting any task.

Key categories your constitution should cover (for quick audit):

| Category | Covered in CONSTITUTION.md? |
|------|-------------------------------|
| Naming conventions | ✅ / ❌ |
| Architecture rules (layer boundaries) | ✅ / ❌ |
| Error handling (exception types, handler) | ✅ / ❌ |
| Logging (PII rules, log levels) | ✅ / ❌ |
| Approved libraries (no new deps without approval) | ✅ / ❌ |
| Security (auth required, no exposed stack traces) | ✅ / ❌ |
| Public contracts (no changes without approval) | ✅ / ❌ |
| Testing (coverage floor, unit + integration) | ✅ / ❌ |

---

## 7. Definition of Done ✅

> A feature is **done-done** only when every box is checked.  
> Full rationale → `LIVING_SPEC.md`

- [ ] `requirements.md` reflects the final behavior — no drift from implementation
- [ ] `design.md` updated with final architecture and any decisions made during implementation
- [ ] All tasks in `tasks.md` are completed and verified
- [ ] AI instruction files (`CLAUDE.md` / `AGENTS.md`) are current
- [ ] All acceptance criteria pass (automated + manual)
- [ ] All Constitution rules are respected
- [ ] No unauthorized files were modified
- [ ] No unresolved `[NEEDS_CLARIFICATION]` items
- [ ] Zero unresolved TODOs or FIXMEs left in changed files
- [ ] PR merged and linked to task IDs

---

## 8. Audit Trail

> Every change must be traceable end-to-end.

```
Requirement (SDD)
    └── Task (tasks.md) ── T-XX
            └── Pull Request ── PR #[N]
                    └── Acceptance Criteria ── AC-XX / SEC-XX
```

| Task ID | PR # | Criteria Covered | Merged By | Date |
|---------|------|-----------------|-----------|------|
| T-01 | #[N] | AC-01, AC-02 | [Name] | YYYY-MM-DD |
| T-02 | #[N] | SEC-01 | [Name] | YYYY-MM-DD |
