# hooks/IMPLEMENTATION.md — How to Wire Agent Hooks

> The hooks in [`HOOKS.md`](HOOKS.md) are defined as behavioral rules.
> This file shows how to implement them using the tools available in your environment.

---

## Three Implementation Targets

Hooks can be wired at three levels depending on how much enforcement you need:

| Level | Tool | Coverage |
|---|---|---|
| **Git** | Git hooks (`pre-commit`, `pre-push`) | Catches violations before commits/pushes |
| **Editor** | VS Code tasks + Claude Code hooks | Catches violations during active development |
| **CI** | GitHub Actions | Catches violations before merging |

Most teams implement Never hooks in git + CI, Ask hooks in the editor, Always hooks in the editor.

---

## Level 1 — Git Hooks

Git hooks live in `.git/hooks/` or (better) in a committed `.githooks/` folder with:

```bash
git config core.hooksPath .githooks
```

### Implementing NEVER-001: Block commit of secrets

`.githooks/pre-commit`:
```bash
#!/bin/bash

# Scan staged files for secret patterns
PATTERNS=(
  '(?i)(api_key|secret|password|token|private_key)\s*=\s*['"'"'"][^'"'"'"]{8,}'
  'AKIA[0-9A-Z]{16}'
  'sk-[a-zA-Z0-9]{32,}'
)

STAGED_FILES=$(git diff --cached --name-only)

for file in $STAGED_FILES; do
  for pattern in "${PATTERNS[@]}"; do
    if git show ":$file" | grep -P "$pattern" > /dev/null 2>&1; then
      echo "BLOCKED: Potential secret detected in $file"
      echo "Pattern matched: $pattern"
      echo "Remove the secret and try again."
      exit 1
    fi
  done
done

exit 0
```

---

### Implementing NEVER-002: Block .env files

`.githooks/pre-commit` (add to the script above):
```bash
# Block .env files
for file in $STAGED_FILES; do
  if echo "$file" | grep -E '(^|/)\.(env)(\.|$)' > /dev/null 2>&1; then
    echo "BLOCKED: .env files must never be committed."
    echo "Add $file to .gitignore"
    exit 1
  fi
done
```

---

### Implementing NEVER-003: Block direct push to main

`.githooks/pre-push`:
```bash
#!/bin/bash

REMOTE=$1
URL=$2

while read local_ref local_sha remote_ref remote_sha; do
  if [[ "$remote_ref" == "refs/heads/main" || "$remote_ref" == "refs/heads/master" ]]; then
    echo "BLOCKED: Direct push to main is not allowed."
    echo "Open a pull request instead."
    exit 1
  fi
done

exit 0
```

---

### Implementing ALWAYS-002: Validate migration file naming

`.githooks/pre-commit` (add to the script):
```bash
# Validate Flyway migration naming
for file in $STAGED_FILES; do
  if echo "$file" | grep -E 'db/migration/.+\.sql$' > /dev/null 2>&1; then
    filename=$(basename "$file")
    if ! echo "$filename" | grep -E '^V[0-9]+__[A-Za-z0-9_]+\.sql$' > /dev/null 2>&1; then
      echo "BLOCKED: Migration file name invalid: $filename"
      echo "Expected format: V{n}__{Description}.sql  (e.g. V5__Add_expiry_column.sql)"
      exit 1
    fi
  fi
done
```

---

## Level 2 — Claude Code Hooks (Editor)

Claude Code supports hooks that run automatically during AI sessions. Configure in `.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "bash .githooks/pre-edit-check.sh \"$CLAUDE_TOOL_INPUT_FILE_PATH\""
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "bash .githooks/post-edit-lint.sh \"$CLAUDE_TOOL_INPUT_FILE_PATH\""
          }
        ]
      }
    ]
  }
}
```

### Implementing ALWAYS-001: Run linter on save (Claude Code)

`.githooks/post-edit-lint.sh`:
```bash
#!/bin/bash
FILE=$1

if echo "$FILE" | grep -E '\.java$' > /dev/null 2>&1; then
  ./mvnw checkstyle:check -q 2>&1
  if [ $? -ne 0 ]; then
    echo "Checkstyle violations found. Fix before continuing."
    exit 1
  fi
fi

exit 0
```

### Implementing ASK-002: Detect out-of-scope file edits (Claude Code)

`.githooks/pre-edit-check.sh`:
```bash
#!/bin/bash
FILE=$1
TASKS_FILE="specs/active/$(ls specs/active/ 2>/dev/null | head -1)/tasks.md"

if [ -f "$TASKS_FILE" ]; then
  # Extract files listed in the current task
  TASK_FILES=$(grep -A 5 '### TASK' "$TASKS_FILE" | grep '^\- \`' | sed 's/- `//;s/`//')

  IN_SCOPE=false
  for task_file in $TASK_FILES; do
    if echo "$FILE" | grep -q "$task_file"; then
      IN_SCOPE=true
      break
    fi
  done

  if [ "$IN_SCOPE" = false ]; then
    echo "WARNING: $FILE is not listed in the current task scope."
    echo "Current task files: $TASK_FILES"
    echo "Proceed only if this change is intentional."
    # Exit 0 to warn but not block (Ask tier, not Never tier)
    exit 0
  fi
fi

exit 0
```

---

## Level 3 — GitHub Actions (CI)

CI hooks catch anything that slipped past local hooks and act as the final safety net.

### Implementing NEVER-001 + NEVER-002 in CI

`.github/workflows/security-gate.yml`:
```yaml
name: Security Gate

on: [push, pull_request]

jobs:
  secret-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Scan for secrets
        uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          base: ${{ github.event.repository.default_branch }}
          head: HEAD

      - name: Check for .env files
        run: |
          if find . -name ".env*" -not -path "./.git/*" | grep -q .; then
            echo "ERROR: .env file found in repository"
            find . -name ".env*" -not -path "./.git/*"
            exit 1
          fi

  migration-naming:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Validate migration file names
        run: |
          find . -path "*/db/migration/*.sql" | while read f; do
            filename=$(basename "$f")
            if ! echo "$filename" | grep -qE '^V[0-9]+__[A-Za-z0-9_]+\.sql$'; then
              echo "Invalid migration file name: $filename"
              exit 1
            fi
          done
```

---

## Quick Reference — Hook to Implementation

| Hook ID | Tier | Implementation target | Tool |
|---|---|---|---|
| ALWAYS-001 | Always | Claude Code PostToolUse | `post-edit-lint.sh` |
| ALWAYS-002 | Always | Git pre-commit | Shell script |
| ALWAYS-003 | Always | Claude Code PostToolUse | Append timestamp |
| ASK-001 | Ask | Git pre-commit (warn) | Shell script |
| ASK-002 | Ask | Claude Code PreToolUse | `pre-edit-check.sh` |
| ASK-003 | Ask | GitHub Actions | Coverage check step |
| NEVER-001 | Never | Git pre-commit + CI | Shell + TruffleHog |
| NEVER-002 | Never | Git pre-commit + CI | Shell + Actions |
| NEVER-003 | Never | Git pre-push | Shell script |
| NEVER-004 | Never | CI | Check merged SPEC PR status |
