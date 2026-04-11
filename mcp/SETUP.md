# mcp/SETUP.md — Configuring MCP Servers

> How to install and connect the three MCP servers used in this topic.
> Takes about 15 minutes from zero to your first diagnostic session.

---

## Prerequisites

- Node.js 18+ installed
- Claude Desktop or Claude Code CLI
- Chrome or Chromium installed (for Playwright + DevTools)
- A Figma account with API access (for Figma MCP, optional)

---

## Step 1 — Install the MCP Servers

```bash
# Playwright MCP server
npm install -g @playwright/mcp

# Chrome DevTools MCP server
npm install -g @chrome-devtools/mcp

# Figma MCP server (optional)
npm install -g @figma/mcp
```

---

## Step 2 — Configure Claude Desktop

Claude Desktop reads its MCP configuration from `claude_desktop_config.json`.

**Location:**
- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`

**Minimal config (Playwright + DevTools):**
```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp", "--browser", "chromium"],
      "env": {
        "PLAYWRIGHT_HEADLESS": "true"
      }
    },
    "devtools": {
      "command": "npx",
      "args": ["@chrome-devtools/mcp"],
      "env": {
        "CDP_PORT": "9222"
      }
    }
  }
}
```

**With Figma:**
```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp", "--browser", "chromium"]
    },
    "devtools": {
      "command": "npx",
      "args": ["@chrome-devtools/mcp"]
    },
    "figma": {
      "command": "npx",
      "args": ["@figma/mcp"],
      "env": {
        "FIGMA_ACCESS_TOKEN": "your-figma-personal-access-token"
      }
    }
  }
}
```

Restart Claude Desktop after editing this file.

---

## Step 3 — Configure Claude Code (CLI)

Claude Code reads MCP config from `.claude/settings.json` in your project root, or from `~/.claude/settings.json` globally.

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp", "--browser", "chromium"],
      "env": {
        "PLAYWRIGHT_BASE_URL": "http://localhost:3000"
      }
    },
    "devtools": {
      "command": "npx",
      "args": ["@chrome-devtools/mcp"]
    }
  }
}
```

---

## Step 4 — Start Your Dev Server

MCP sessions run against **local dev only** (see [`governance/BOUNDED_AUTONOMY.md`](governance/BOUNDED_AUTONOMY.md)).

Start your app before opening Claude:

```bash
# Java / Spring Boot
./mvnw spring-boot:run

# Node / Next.js
npm run dev

# Docker Compose
docker-compose up
```

---

## Step 5 — Verify the Connection

Open Claude Desktop or run `claude` in your terminal. Ask:

```
List the MCP tools available in this session.
```

You should see tools from each configured server:

```
playwright:
  - navigate
  - click
  - screenshot
  - evaluate
  - fill
  - waitForSelector

devtools:
  - startTrace
  - stopTrace
  - getNetworkLog
  - getConsoleLog
  - queryTrace
```

If tools are missing, check the server names in `claude_desktop_config.json` match what the npm package exports.

---

## Step 6 — Run Your First Diagnostic

Open a Claude session with your servers active. Paste your spec requirement from `example-diagnostic/spec-requirement.md` and say:

```
Using the Playwright and DevTools MCP servers, run the Runtime Gateflow
for this requirement. Start by navigating to http://localhost:3000/products
and capturing a performance trace.
```

The agent will call `startTrace`, navigate, `stopTrace`, and return structured evidence you can log in `evidence-log.md`.

---

## Enabling DevTools Remote Protocol

For the DevTools MCP server to work, Chrome must be launched with remote debugging enabled.

**Option A — Launch Chrome manually:**
```bash
# macOS
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
  --remote-debugging-port=9222 \
  --user-data-dir=/tmp/chrome-debug

# Windows
chrome.exe --remote-debugging-port=9222 --user-data-dir=C:\tmp\chrome-debug

# Linux
google-chrome --remote-debugging-port=9222 --user-data-dir=/tmp/chrome-debug
```

**Option B — Playwright manages the browser (recommended):**
The Playwright MCP server launches its own Chromium instance with CDP enabled automatically. Use this for most diagnostic sessions.

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| "No MCP tools available" | Restart Claude Desktop; check `claude_desktop_config.json` syntax |
| Playwright can't navigate | Confirm dev server is running at the configured base URL |
| DevTools returns empty trace | Make sure CDP port 9222 is open; use Playwright-managed browser |
| Figma returns 403 | Confirm `FIGMA_ACCESS_TOKEN` is set and the file is shared/published |
| Screenshot is blank | Disable headless mode: remove `PLAYWRIGHT_HEADLESS: "true"` |

---

## Bounded Autonomy Reminder

MCP is powerful. Before each session, confirm:

- [ ] Target is `localhost` or a dedicated test environment — never production
- [ ] No real user credentials used in Playwright `fill()` calls
- [ ] Heap snapshots not enabled unless explicitly needed
- [ ] Tool manifest logged at session start

Full rules: [`governance/BOUNDED_AUTONOMY.md`](governance/BOUNDED_AUTONOMY.md)
