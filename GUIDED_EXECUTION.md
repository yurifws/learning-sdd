# 🤖 Guided Execution — `[Project Name]`

> The AI is a **guided implementer**, not a free agent.  
> Every change must trace back to a task. Every task must trace back to the plan.

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
| AC-01 | [trigger] | [behavior] | `mvn test -Dtest=[TestClass]` |
| AC-02 | [trigger] | [behavior] | `curl -X POST ...` returns HTTP 4xx |
| AC-03 | [trigger] | [behavior] | Manual checklist item #[N] |

### 🔴 Negative / Security Criteria

| # | To prevent... | The system SHALL... | Verified by |
|---|--------------|---------------------|-------------|
| SEC-01 | User enumeration | Return identical response for existing and non-existing emails | `[TestClass]#shouldNotLeakUserExistence` |
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

**PR title format:** `[T-XX] Short description of what was done`  
**PR description must include:**
- Task ID reference (e.g., `Closes T-07`)
- Verify command output or test results
- Link back to SDD / Constitution section

---

## 4. Drift Detection — Red Flags 🚩

> If any of these appear in a PR, it is a **hard stop**. Update the plan first.

| Red Flag | What It Means | Action |
|----------|--------------|--------|
| New library not in Constitution | AI went off-plan | Reject PR, update plan if needed |
| Files touched not listed in task | Scope creep | Reject PR, split into smaller tasks |
| Edge case fixed that wasn't specified | Unauthorized change | Reject PR, create a new task for it |
| Refactor outside task scope | Classic drift | Reject PR, separate refactor task |
| Behavior change with no task reference | Untraceable change | Reject PR immediately |

---

## 5. Constitution Reference

> Non-negotiable rules. Paste key constraints here for quick AI reference.

| Rule | Constraint |
|------|-----------|
| Naming | [e.g., camelCase methods, PascalCase classes] |
| Architecture | [e.g., no business logic in controllers] |
| Error handling | [e.g., always use GlobalExceptionHandler] |
| Logging | [e.g., never log passwords, tokens, or PII] |
| Libraries | [e.g., only approved deps in pom.xml] |
| Security | [e.g., all endpoints require auth unless explicitly public] |

> Full rules → `CONSTITUTION.md`

---

## 6. Definition of Done ✅

> A feature is **done-done** only when every box is checked.

- [ ] All tasks in `tasks.md` are completed and verified
- [ ] All acceptance criteria pass (automated + manual)
- [ ] All Constitution rules are respected
- [ ] No unauthorized files were modified
- [ ] `SDD.md` updated to reflect what was actually built
- [ ] `tasks.md` fully checked off
- [ ] Zero unresolved TODOs or FIXMEs left in changed files
- [ ] PR merged and linked to task IDs

---

## 7. Audit Trail

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
