# governance/BOUNDED_AUTONOMY.md

> The agent is not free to roam your system. It can only call the specific tools
> that you have explicitly declared and given it permission to use.
>
> This is what makes MCP safe for real projects.

---

## The Three-Tier Rule Set

The same Always / Ask / Never pattern from `kiro/hooks/HOOKS.md` applies here —
but scoped to live-system access through MCP servers.

---

## Tier 1 — Always

Actions that run automatically as part of every diagnostic session. No confirmation required.

These are safe, read-only, and produce no side effects.

---

### ALWAYS-001 · Record all evidence and link it to the spec

Every observation the agent makes — a screenshot, a trace result, a network log — must be:
1. Saved to `example-diagnostic/evidence-log.md`
2. Tagged with the EARS clause it was gathered for

**Why:** Evidence that isn't linked to a requirement is useless. The spec is the anchor.

```yaml
trigger: mcp.tool.response
action: append_to_evidence_log
fields: [tool, args, response_summary, spec_clause_id]
```

---

### ALWAYS-002 · Use sandbox / local environment only

All MCP server connections default to local dev or a dedicated test environment.

The agent never needs to ask — it just never connects to production without an explicit override.

```yaml
trigger: mcp.session.start
action: assert_environment
allowed: [localhost, *.test, *.staging]
on_violation: block_and_alert
```

---

### ALWAYS-003 · Log tool declarations before session starts

Before the diagnostic session begins, log the full list of tools and permissions
granted to the agent for this session.

**Why:** Full auditability. Anyone reviewing the session can see exactly what the agent was allowed to do.

```yaml
trigger: mcp.session.start
action: log_tool_manifest
destination: evidence-log.md
```

---

## Tier 2 — Ask

Actions that require explicit human confirmation before proceeding.
The session is paused until the user approves or denies.

These actions are meaningful, potentially state-changing, or involve elevated access.

---

### ASK-001 · Any action that modifies application state

If a Playwright action would submit a form, click a "Delete" button, or trigger any write operation:

```yaml
trigger: playwright.action
condition: action_type in [click_submit, fill_and_submit, delete_trigger]
action: ask
message: |
  About to perform a write action in the browser:
    Action: {action_type}
    Target: {selector}
    Environment: {env}
  
  This may create or modify data. Proceed? (yes/no)
on_deny: skip_action
```

---

### ASK-002 · Heap snapshot capture

Heap snapshots may contain sensitive in-memory state (user tokens, PII, cached API responses).

```yaml
trigger: devtools.tool_call
condition: tool == "getHeapSnapshot"
action: ask
message: |
  A heap snapshot captures all in-memory data, including potentially sensitive state.
  Environment: {env}
  
  Capture heap snapshot? (yes/no)
on_deny: skip_tool_call
```

---

### ASK-003 · Expanding tool permissions mid-session

If a diagnostic reveals the need for a tool not declared in the initial session manifest:

```yaml
trigger: mcp.tool.not_in_manifest
action: ask
message: |
  The agent is requesting access to a tool not declared for this session:
    Tool: {tool_name}
    Server: {server_name}
    Reason: {agent_stated_reason}
  
  Grant access for this session? (yes/no)
on_deny: block_tool_call
```

---

## Tier 3 — Never

Hard blocks. The action is forbidden and will not execute under any circumstances.
No override possible within a session.

---

### NEVER-001 · No access to production systems

The MCP session must never connect to a production environment — not for reads, not for traces, not for screenshots.

```yaml
trigger: mcp.session.start OR mcp.tool.call
condition: target_environment == "production"
action: block_and_alert
message: "MCP sessions must never connect to production. Use a local or staging environment."
```

**Why:** Production traces may contain real user data. Screenshots may expose PII. Network logs may expose authentication tokens in headers.

---

### NEVER-002 · No secrets in tool call arguments

Any tool call argument that matches a known secret pattern is blocked before it leaves the agent.

```yaml
trigger: mcp.tool.call
action: scan_arguments
patterns:
  - "(?i)(api_key|secret|password|token|private_key)\\s*=\\s*['\"][^'\"]{8,}"
  - "Bearer [A-Za-z0-9\\-._~+/]+=*"
on_match: block_and_alert
message: "Tool call blocked: argument appears to contain a secret. Use test credentials only."
```

---

### NEVER-003 · No writing to Figma or DevTools

Figma MCP and DevTools MCP are strictly read-only. The agent may never:
- Publish, edit, or duplicate Figma files
- Execute arbitrary JavaScript via DevTools `Runtime.evaluate` with write side effects
- Modify browser state outside the Playwright-controlled session

```yaml
trigger: figma.tool.call OR devtools.tool.call
condition: tool in WRITE_TOOLS
action: block_and_alert
message: "Write operations are not permitted on Figma or DevTools MCP servers."
```

---

### NEVER-004 · No exfiltration of evidence outside the session

Evidence gathered during a diagnostic session (screenshots, traces, heap snapshots) must stay in the local `evidence-log.md`. It must never be sent to an external service, pasted into a public channel, or uploaded anywhere.

```yaml
trigger: any.outbound.request
condition: payload_contains_evidence_artifact
action: block_and_alert
message: "Evidence artifacts must not leave the local diagnostic session."
```

---

## Why This Matters

Without bounded autonomy, MCP is dangerous. With it, the agent becomes a controlled, auditable partner.

```
No boundaries                         Bounded autonomy
────────────────────────────────      ─────────────────────────────────────
Agent connects to production          Agent connects to localhost only
Agent runs arbitrary JS               Agent calls declared tools only
Evidence goes anywhere                Evidence stays in evidence-log.md
No audit trail                        Full tool manifest logged at session start
```

The three tiers give your team a simple mental model: **safe by default, ask for anything meaningful, block everything dangerous.**
