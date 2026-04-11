# servers/playwright.md — Playwright MCP Server

> **Role:** The agent's eyes and hands for the frontend.
> Gives the AI the ability to control a real browser, re-enact user journeys, and capture visual evidence.

---

## What It Does

The Playwright MCP server exposes browser automation as a set of callable tools. The AI agent uses these tools to physically verify user experience — not by reading code, but by running it.

```
Agent                          Playwright MCP Server           Browser
  │                                    │                          │
  │──── navigate("/products") ────────►│                          │
  │                                    │──── page.goto() ────────►│
  │◄─── { status: 200, title: "..." } ─│◄─── DOM loaded ──────────│
  │                                    │                          │
  │──── screenshot() ─────────────────►│                          │
  │◄─── { image: base64... } ──────────│◄─── captured ────────────│
  │                                    │                          │
  │──── click("#add-to-cart") ────────►│                          │
  │◄─── { success: true } ─────────────│◄─── click event ─────────│
```

---

## Available Tools

| Tool | What it does | Typical use |
|---|---|---|
| `navigate(url)` | Opens a URL in the browser | Start a user journey at the right entry point |
| `click(selector)` | Clicks a DOM element | Trigger buttons, links, form submissions |
| `fill(selector, value)` | Types into an input field | Test form flows, search, login |
| `screenshot()` | Captures the current page as an image | Visual evidence — "what the user actually sees" |
| `evaluate(script)` | Runs JavaScript in the page context | Read DOM state, check computed values |
| `waitForSelector(selector)` | Pauses until an element appears | Handle async rendering, loading states |
| `getNetworkRequests()` | Returns the list of HTTP requests made | Check what the page fetched and what it got back |

---

## Example: Reproducing a Bug from a Spec Requirement

**Spec requirement (EARS):**
```
WHEN the user navigates to /products,
  IF the product list is empty,
  THEN the system SHALL display the message "No products found."
  AND SHALL NOT display a loading spinner indefinitely.
```

**Playwright MCP session:**
```json
[
  { "tool": "navigate", "args": { "url": "http://localhost:3000/products?empty=true" } },
  { "tool": "waitForSelector", "args": { "selector": ".product-list", "timeout": 3000 } },
  { "tool": "screenshot", "args": {} },
  { "tool": "evaluate", "args": {
      "script": "document.querySelector('.empty-state')?.textContent"
    }
  }
]
```

**Evidence captured:**
```json
{
  "screenshot": "base64:...",
  "evaluate_result": null,
  "dom_state": "spinner visible, empty-state not present after 3000ms"
}
```

**Conclusion:** Requirement violated. The spinner never stops. The `empty-state` element is never rendered. Root cause investigation hands off to DevTools.

---

## How to Use in the Runtime Gateflow

1. **Step 1 — Reproduce:** Use `navigate` + `click` / `fill` to re-enact the exact user journey described in the spec requirement.
2. **Step 2 — Capture:** Use `screenshot` + `evaluate` to record what the user actually sees. This is the visual evidence.
3. **Step 3 — Hand off:** If the issue is a runtime error or performance problem, pass the session timeline to the Chrome DevTools MCP server for deeper analysis.

---

## Bounded Autonomy Constraints

Playwright is a powerful tool. These constraints apply at all times:

| Rule | Detail |
|---|---|
| **Sandbox only** | Playwright sessions run against local dev or a dedicated test environment — never production |
| **Read-first** | Navigation and observation actions do not require confirmation |
| **Ask before write** | Any action that creates, modifies, or deletes data (form submit, delete button) requires explicit approval |
| **No credentials in scripts** | `fill()` must never receive production passwords or API keys — use test credentials only |

See [`governance/BOUNDED_AUTONOMY.md`](../governance/BOUNDED_AUTONOMY.md) for the full rule set.
