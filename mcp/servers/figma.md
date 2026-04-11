# servers/figma.md — Figma MCP Server

> **Role:** The agent's connection to the design source of truth.
> Ends the "is this the right color?" back-and-forth by giving the AI direct access to design artifacts as queryable data.

---

## What It Does

The Figma MCP server exposes Figma documents as structured, queryable data. Instead of inferring design intent from a text description or a screenshot, the agent reads the actual design file — tokens, spacing, colors, component states — and compares them directly against what the UI renders.

```
Without Figma MCP                    With Figma MCP
─────────────────────────────────    ──────────────────────────────────────
"The button should be blue"          { "fill": "#1A56DB", "radius": 6,
→ AI guesses what blue means           "padding": "10px 16px",
→ Dev guesses if the guess is right    "font-weight": 600 }
→ Designer reviews, finds drift      → AI compares to rendered button
→ Figma re-check, fix, re-review     → Drift detected at pixel level instantly
```

---

## Available Tools

| Tool | What it does | Typical use |
|---|---|---|
| `getNode(nodeId)` | Returns all properties of a specific Figma node | Inspect a component, frame, or text layer |
| `getStyles()` | Returns all design tokens (colors, typography, spacing) | Verify the full token set is applied correctly in code |
| `getComponent(name)` | Returns all variants and states of a component | Check every state (hover, disabled, error) is implemented |
| `compareToScreenshot(nodeId, image)` | Diffs a Figma frame against a screenshot | Visual parity check — quantified, not eyeballed |
| `getAnnotations(nodeId)` | Returns designer notes and redline specs | Surface intent that isn't visible in the final output |
| `getVariables()` | Returns Figma Variables (design tokens as code-ready values) | Bridge between design system and implementation |

---

## Example: Detecting Visual Drift

**Spec requirement (EARS):**
```
WHEN the user views the product card,
  the system SHALL render the "Add to Cart" button
  using the primary-action design token from the design system.
```

**Figma MCP session:**
```json
[
  { "tool": "getComponent", "args": { "name": "Button/Primary" } },
  { "tool": "getStyles", "args": { "scope": "color-tokens" } }
]
```

**Design source of truth:**
```json
{
  "component": "Button/Primary",
  "fill": "#1A56DB",
  "border_radius": 6,
  "padding": "10px 16px",
  "font_size": 14,
  "font_weight": 600,
  "states": {
    "hover": { "fill": "#1E429F" },
    "disabled": { "fill": "#C3DDFD", "cursor": "not-allowed" }
  }
}
```

**Playwright screenshot + evaluate:**
```json
{
  "computed_style": {
    "background-color": "rgb(37, 99, 235)",
    "border-radius": "4px",
    "font-weight": "500"
  }
}
```

**Drift detected:**
```
border-radius: design says 6px  →  rendered as 4px   ✗
font-weight:   design says 600  →  rendered as 500   ✗
fill:          design says #1A56DB  →  rendered as #2563EB  ✗  (Tailwind blue-600, not token)
```

**Conclusion:** The button uses Tailwind utility classes directly instead of the design token. Three visual spec violations. Root cause: `ProductCard.tsx` hardcodes `bg-blue-600` instead of using the `btn-primary` token class.

---

## How to Use in the Runtime Gateflow

1. **Before Playwright:** Pull the Figma spec for the component under test so the agent knows exactly what to verify.
2. **After Playwright screenshot:** Use `compareToScreenshot` or evaluate CSS values and diff against Figma data.
3. **Map to spec:** Every visual drift is a violation of the EARS clause that references the design system. Name it and log it in `evidence-log.md`.
4. **Update spec if needed:** If the design changed in Figma but the EARS requirement wasn't updated, that's a spec gap — fix it.

---

## Figma as the Visual Spec

The Figma MCP server is most powerful when your team treats the Figma document as part of the spec, not separate from it. Every EARS clause that describes UI behavior should reference a Figma node ID or component name.

```
WHEN the user views a product card,
  the system SHALL render the "Add to Cart" button
  matching the "Button/Primary" component in Figma node #4821-099.
```

This makes verification mechanical, not subjective.

---

## Bounded Autonomy Constraints

| Rule | Detail |
|---|---|
| **Read-only** | Figma MCP only reads design data — it never writes, publishes, or modifies Figma files |
| **Published files only** | Connect only to shared/published Figma files, never to personal drafts |
| **No PII in annotations** | Designer annotations may contain user research notes — the agent must not log or surface these outside the diagnostic session |
