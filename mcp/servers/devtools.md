# servers/devtools.md — Chrome DevTools MCP Server

> **Role:** The agent's access to runtime internals.
> Gives the AI the same diagnostic tools developers use every day — performance traces, network logs, JS timelines — as structured, queryable evidence.

---

## What It Does

The Chrome DevTools MCP server exposes the Chrome DevTools Protocol (CDP) as callable tools. The AI agent uses these to go beyond what you can see on screen — into what the browser is actually doing at runtime.

This is what closes the gap between a vague requirement like:

> "The page should be fast."

and an actionable piece of evidence:

> "The `fetchProducts()` call takes 1,847ms. It fires on every keystroke in the search box because the debounce function is missing. Here is the flame graph."

---

## Available Tools

| Tool | What it does | Typical use |
|---|---|---|
| `startTrace()` | Begins a performance recording | Capture everything from navigation to first interaction |
| `stopTrace()` | Stops the recording and returns the trace | Get the full timeline as structured JSON |
| `getNetworkLog()` | Returns all HTTP requests with timing | Find slow API calls, missing caches, waterfall bottlenecks |
| `getConsoleLog()` | Returns all console.log / error / warn output | Surface JS errors that don't crash the page visibly |
| `getCoverage()` | Returns JS/CSS code coverage | Find dead code loaded but never executed |
| `getHeapSnapshot()` | Returns a memory snapshot | Detect memory leaks in long-running sessions |
| `queryTrace(filter)` | Filters a trace by function name or duration | Find the specific slow function without reading the full flame graph |

---

## Example: Diagnosing a Performance Violation

**Spec requirement (EARS):**
```
WHEN the user navigates to /products,
  the system SHALL render the full product list within 1,500ms
  as measured from the first byte of the HTML response.
```

**DevTools MCP session:**
```json
[
  { "tool": "startTrace", "args": { "categories": ["loading", "rendering", "scripting"] } },
  // <Playwright navigates to /products here>
  { "tool": "stopTrace", "args": {} },
  { "tool": "queryTrace", "args": { "filter": { "duration_gt_ms": 200 } } },
  { "tool": "getNetworkLog", "args": {} }
]
```

**Evidence captured:**
```json
{
  "slow_functions": [
    { "name": "fetchProducts", "duration_ms": 1847, "file": "api/products.js", "line": 42 },
    { "name": "renderProductCard", "duration_ms": 312, "called_times": 200 }
  ],
  "network_log": [
    { "url": "/api/v1/products", "duration_ms": 1623, "status": 200, "size_kb": 842 }
  ]
}
```

**Conclusion:** Two violations:
1. `/api/v1/products` returns 842KB of data — no pagination applied on the API call.
2. `renderProductCard` called 200 times synchronously — list is not virtualized.

Total render time: 2,159ms. Requirement says 1,500ms. **Requirement violated. Root cause localized.**

---

## Reading a Trace Result

```
fetchProducts (1847ms)          ← main bottleneck
  └── fetch() (1623ms)          ← network call to /api/v1/products
      └── JSON.parse (224ms)    ← 842KB payload parsed synchronously

renderProductCard × 200 (312ms total)
  └── createElement × 200       ← no virtualization, all 200 rendered at once
```

The agent maps each entry to a file and line number, then proposes a fix with evidence of what changed.

---

## How to Use in the Runtime Gateflow

1. **Start trace before Playwright navigation** — capture the full timeline from the very beginning.
2. **Use `queryTrace` to filter noise** — ask only for functions slower than a threshold.
3. **Cross-reference with `getNetworkLog`** — slow renders are often caused by slow API calls, not slow rendering code.
4. **Map findings to spec** — every slow function is evidence against a specific EARS clause. Name it.

---

## Bounded Autonomy Constraints

| Rule | Detail |
|---|---|
| **Read-only** | DevTools MCP only reads runtime data — it never executes code or modifies application state |
| **Sandbox only** | Connect only to local dev or dedicated test browser instances |
| **No production profiling** | Never attach to a production Chrome instance — traces may contain user PII |
| **Heap snapshots require approval** | Heap data can contain sensitive in-memory state — ask before capturing |
