
## Approach

1. Clone & build the MCP server locally (instead of npx fetching it)
2. Point VS Code's `mcp.json` to that local build
3. Set up a real Playwright test framework in the repo (Page Object Model etc.) so Copilot Chat has context
4. Add a `copilot-instructions.md` so every chat query follows your framework's conventions
5. Ask Copilot Chat (Agent mode) to generate tests — it drives a real browser via your local MCP server and writes scripts into your framework

## Step 1: Clone and build Playwright MCP locally

```bash
git clone https://github.com/microsoft/playwright-mcp.git
cd playwright-mcp
npm install
npm run build
npx playwright install   # installs browser binaries
```

This gives you `playwright-mcp/cli.js` (or `lib/index.js` depending on version) that runs standalone with `node`, no npm registry call needed at runtime.

## Step 2: Point VS Code to the local build

Create `.vscode/mcp.json` in your test-framework project:

```json
{
  "servers": {
    "playwright-local": {
      "command": "node",
      "args": [
        "C:/tools/playwright-mcp/cli.js",
        "--browser", "chromium",
        "--headless=false"
      ]
    }
  }
}
```

Reload VS Code → Command Palette → `MCP: List Servers` → start `playwright-local`. It should show as connected in Copilot Chat's tools panel.

## Step 3: Set up your Playwright framework (if not already done)

```bash
npm init playwright@latest
```

Structure it so Copilot can reuse it:

```
/tests
  /pages          (Page Object classes)
  /fixtures       (custom test fixtures)
  /specs          (actual .spec.ts files)
playwright.config.ts
```

## Step 4: Give Copilot Chat your framework as context

Create `.github/copilot-instructions.md` in the repo root:

```markdown
# Test Framework Conventions
- Use TypeScript and Playwright Test.
- Follow Page Object Model: put locators/actions in /tests/pages.
- New specs go in /tests/specs, one file per feature.
- Use existing fixtures from /tests/fixtures instead of raw `page` where available.
- Prefer role-based locators (getByRole, getByLabel) over CSS selectors.
```

VS Code Copilot automatically loads this file as context for every chat request.

## Step 5: Use it

In Copilot Chat, switch to **Agent mode**, then ask something like:

> "Open https://myapp.local, log in with the test user, add an item to cart, and write a Playwright spec for it following our framework conventions."

What happens:
- Copilot calls your **local** MCP server tools (`browser_navigate`, `browser_click`, `browser_snapshot`, etc.)
- It actually drives the browser and inspects the accessibility tree
- Using `copilot-instructions.md` as context, it generates a `.spec.ts` file following your POM structure and drops it in `/tests/specs`

A few practical notes worth passing to your manager:
- Local build means you control the version and it works offline, but you're responsible for updating it (`git pull && npm install && npm run build`) — no auto-updates like `npx @playwright/mcp@latest`.
- Microsoft's newer guidance suggests a companion **CLI+Skills** approach uses far fewer tokens than MCP for coding-agent-heavy workflows — worth a look if context/token cost becomes a concern, but MCP is the right choice for interactive chat-driven test generation like this.
- The `mcp.json` can be committed to the repo (`.vscode/mcp.json`) so the whole team shares the same local server config — they just each need `playwright-mcp` cloned at the same path (or use a workspace-relative path).

Want me to generate an actual starter repo structure (config files, sample page object, sample spec) as downloadable files?
