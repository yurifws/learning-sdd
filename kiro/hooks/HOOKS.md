# hooks/HOOKS.md

> Agent hooks are automated rules tied to editor events.
> They turn team standards from documentation nobody reads into enforcement nobody can skip.
>
> Three tiers: **Always** (runs automatically) · **Ask** (requests permission first) · **Never** (hard block)

---

## Tier 1 — Always

Runs automatically every time the trigger fires. No prompt, no approval.

These are low-risk, high-value actions that should happen unconditionally.

---

### ALWAYS-001 · Run linter on save

**Trigger:** Any `.java` file is saved  
**Action:** Run `./mvnw checkstyle:check -q` on the saved file  
**Why:** Catches style violations immediately — never let them accumulate to PR time  

```yaml
# .kiro/hooks/always-lint-on-save.yaml
trigger: file.saved
glob: "**/*.java"
action: run
command: "./mvnw checkstyle:check -q"
on_failure: block_save
```

---

### ALWAYS-002 · Validate migration file naming

**Trigger:** A new file is created inside `src/main/resources/db/migration/`  
**Action:** Assert filename matches `V{number}__{Description}.sql`  
**Why:** Flyway silently ignores malformed migration names — this catches it at creation time  

```yaml
trigger: file.created
glob: "src/main/resources/db/migration/*.sql"
action: assert
pattern: "^V[0-9]+__[A-Za-z0-9_]+\\.sql$"
on_failure: block_and_message
message: "Migration file must follow the pattern V{n}__{Description}.sql"
```

---

### ALWAYS-003 · Sync tasks.md after each completed task

**Trigger:** A task in `tasks.md` is marked `[x]`  
**Action:** Append a completion timestamp to the task line  
**Why:** Creates a timestamped audit trail of what the AI did and when  

```yaml
trigger: file.saved
glob: "**/tasks.md"
action: append_timestamp_to_checked_tasks
```

---

### ALWAYS-004 · Run property tests before task completion

**Trigger:** A task in `tasks.md` is about to be marked `[x]`  
**Action:** Run the property test suite for the feature. If any property fails, block the completion and report the minimal counterexample.  
**Why:** A task is not done if it violates a system invariant. Example tests can pass while a property fails — coverage does not equal correctness.

```yaml
trigger: task.before_complete
condition: property_tests_exist_for_feature
action: run
# Adapt command to your build system:
#   Java (Maven/jqwik):  ./mvnw test -Dgroups=property -pl {feature_module}
#   Python (Hypothesis): pytest tests/ -k "property"
#   TypeScript (FastCheck): npm test -- --testNamePattern="property"
command: "./mvnw test -Dgroups=property -pl {feature_module}"
on_failure: block_completion_and_message
message: |
  Property test failed before task was marked complete.
  Hypothesis/jqwik found a counterexample:

    {failing_property}: {minimal_counterexample}

  Fix the invariant violation before closing this task.
```

**What this blocks:**
- Implementations that pass all example tests but fail on boundary inputs
- Security invariants violated by edge-case inputs the AI didn't think to try
- Numeric precision bugs that only appear at `0.01` or `MAX_INT`

---

## Tier 2 — Ask

Pauses and requests explicit user permission before proceeding.
The action is blocked until the user approves or denies.

Use for actions that are reversible but have meaningful side effects.

---

### ASK-001 · New dependency added

**Trigger:** `pom.xml` is modified and a new `<dependency>` block appears  
**Action:** Pause. Display the new dependency. Ask: "Approve adding `{groupId}:{artifactId}:{version}`?"  
**Why:** Prevents dependency sprawl. Every new library must be a conscious team decision.  

```yaml
trigger: file.saved
glob: "pom.xml"
detect: new_xml_element
element: "dependency"
action: ask
message: |
  A new dependency is about to be added:
    {groupId}:{artifactId}:{version}
  
  Is this approved? (yes/no)
on_deny: revert_file
```

---

### ASK-002 · File outside task scope is modified

**Trigger:** A file is saved that is not listed in the current `tasks.md` active task  
**Action:** Pause. Show the unexpected file. Ask: "This file is outside the current task scope — proceed?"  
**Why:** Drift detection. In SDD, any file touched outside the task list is a hard stop.  

```yaml
trigger: file.saved
condition: file_not_in_active_task_scope
action: ask
message: |
  '{filename}' is not listed in the current task scope.
  
  Active task: {task_title}
  Expected files: {task_file_list}
  
  Proceed anyway? (yes/no)
on_deny: revert_file
```

---

### ASK-003 · Test coverage drops below threshold

**Trigger:** Test run completes  
**Action:** If line coverage drops below 80%, pause before proceeding  
**Why:** Coverage regressions are easy to introduce and hard to recover from  

```yaml
trigger: test.run.completed
condition: coverage_below_threshold
threshold: 80
action: ask
message: |
  Test coverage dropped to {coverage}% (threshold: 80%).
  
  Proceed with this coverage? (yes/no)
on_deny: block_commit
```

---

## Tier 3 — Never

Hard blocks. The action is forbidden and will not execute under any circumstances.
No override, no prompt. The AI must stop and report.

Use for actions that are irreversible, security-sensitive, or catastrophically risky.

---

### NEVER-001 · Block commit of secrets

**Trigger:** `git commit` is attempted  
**Action:** Scan staged files for patterns matching API keys, passwords, tokens  
**Why:** Secrets committed to git are almost impossible to fully erase  

```yaml
trigger: git.pre_commit
action: scan_staged_files
patterns:
  - "(?i)(api_key|secret|password|token|private_key)\\s*=\\s*['\"][^'\"]{8,}"
  - "AKIA[0-9A-Z]{16}"               # AWS Access Key
  - "sk-[a-zA-Z0-9]{32,}"            # OpenAI-style key
on_match: block_commit_and_alert
message: "Potential secret detected in staged file '{filename}' at line {line}. Commit blocked."
```

---

### NEVER-002 · Block commit of .env files

**Trigger:** `git commit` is attempted  
**Action:** If any file matching `.env`, `.env.*`, or `*.env` is staged, block  
**Why:** `.env` files almost always contain credentials — there is no safe version of this  

```yaml
trigger: git.pre_commit
action: assert_not_staged
glob: "**/.env*"
on_match: block_commit_and_alert
message: ".env files must never be committed. Add to .gitignore."
```

---

### NEVER-003 · Block direct push to main

**Trigger:** `git push` to `main` or `master` branch  
**Action:** Block immediately  
**Why:** All changes to main must go through a PR — no exceptions  

```yaml
trigger: git.pre_push
condition: target_branch_is_main
action: block_and_message
message: "Direct push to 'main' is not allowed. Open a pull request."
```

---

### NEVER-004 · Block requirements.md modification after SPEC PR is merged

**Trigger:** `requirements.md` inside `specs/active/` is modified  
**Action:** Check if the corresponding SPEC PR is already merged. If yes, block.  
**Why:** Approved requirements are a contract — changing them after approval invalidates all downstream work  

```yaml
trigger: file.saved
glob: "specs/active/**/requirements.md"
condition: spec_pr_already_merged
action: block_and_message
message: |
  requirements.md for this feature is locked — its SPEC PR has already been merged.
  To change requirements: open a new feature branch and create a new SPEC PR.
```

---

## Hook Decision Guide

```
Is the action low-risk and always correct?
  YES → Always hook

Is the action meaningful but reversible, and needs a human judgment call?
  YES → Ask hook

Is the action irreversible, security-sensitive, or violates a hard rule?
  YES → Never hook
```
