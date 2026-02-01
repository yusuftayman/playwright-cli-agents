# Playwright CLI Agents for Claude Code

A production-ready set of **Claude Code agents** and **skills** that automate Playwright E2E test generation, debugging, and planning using the **Page Object Model** pattern.

Drop these files into any project's `.claude/` directory and get an AI-powered E2E testing workflow — from exploring a live URL to generating robust, maintainable test suites.

---

## Getting Started

### Installation

```bash
npm install -g @playwright/cli@latest
playwright-cli --help
```

Note that you might need to force it to reclaim the `playwright-cli` binary from our older MCP package.

```bash
npm install -g @playwright/cli@latest --force
```

### Demo

```
> Use playwright skills to test https://demo.playwright.dev/todomvc/.
  Take screenshots for all successful and failing scenarios.
```

Your agent will be running commands, but it does not mean you can't play with it manually:

```bash
playwright-cli open https://demo.playwright.dev/todomvc/ --headed
playwright-cli type "Buy groceries"
playwright-cli press Enter
playwright-cli type "Water flowers"
playwright-cli press Enter
playwright-cli check e21
playwright-cli check e35
playwright-cli screenshot
```

### Skills-less operation

Point your agent at the CLI and let it cook. It'll read the skill off `playwright-cli --help` on its own:

```
Test the "add todo" flow on https://demo.playwright.dev/todomvc using playwright-cli.
Check playwright-cli --help for available commands.
```

### Installing skills

Claude Code, GitHub Copilot and others will let you install the Playwright skills into the agentic loop.

**Plugin (recommended)**

```bash
/plugin marketplace add microsoft/playwright-cli
/plugin install playwright-cli
```

**Manual**

```bash
mkdir -p .claude/skills/playwright-cli
curl -o .claude/skills/playwright-cli/SKILL.md \
  https://raw.githubusercontent.com/microsoft/playwright-cli/main/skills/playwright-cli/SKILL.md
```

---

## What's Inside

```
.claude-plugin/
└── marketplace.json                     # Plugin marketplace catalog

plugins/
└── playwright-cli-agents/
    ├── .claude-plugin/
    │   └── plugin.json                  # Plugin manifest
    ├── agents/                          # Specialized subagents
    │   ├── playwright-cli-planner.md    # Explores pages → creates test plans
    │   ├── playwright-cli-generator.md  # Executes plans → writes test code
    │   └── playwright-cli-healer.md     # Debugs & fixes failing tests
    └── skills/
        ├── playwright-cli/              # Browser automation via CLI
        │   └── SKILL.md                 # playwright-cli command reference
        └── e2e/                         # Shared domain knowledge
            ├── SKILL.md                 # Core skill (auto-injected into agents)
            ├── examples/
            │   ├── page-object-model.md # Page Object class templates
            │   └── e2e-tests.md         # Integration test patterns
            └── references/
                ├── component-exploration.md # How to explore UIs via playwright-cli
                └── api-mocking.md       # Deterministic test data setup
```

## The Three Agents

| Agent | Purpose | Trigger Example |
|-------|---------|-----------------|
| **Planner** | Navigates to a URL, explores interactive elements, and produces a structured test plan | *"Create a test plan for our checkout flow at localhost:3000/checkout"* |
| **Generator** | Takes a test plan (or creates one), drives the browser in real-time, and outputs Page Object + spec files | *"Generate Playwright tests for the login page"* |
| **Healer** | Runs failing tests, inspects the DOM, console, and network, then fixes broken locators/timing | *"The dashboard test is failing after the redesign, fix it"* |

### How They Work Together

```
 You describe what to test
          │
          ▼
   ┌─────────────┐
   │   Planner    │  Explores the page, identifies elements,
   │   (green)    │  writes a markdown test plan
   └──────┬──────┘
          │ test plan
          ▼
   ┌─────────────┐
   │  Generator   │  Drives browser via playwright-cli, records actions,
   │   (blue)     │  generates Page Object + spec files
   └──────┬──────┘
          │ test files
          ▼
   ┌─────────────┐
   │   Healer     │  Runs tests, catches failures,
   │   (red)      │  auto-fixes locators & timing
   └─────────────┘
```

## The E2E Skill

The `e2e` skill is **shared domain knowledge** that all three agents preload via the `skills: [e2e, playwright-cli]` frontmatter field. It defines:

- **Test file structure** — where Page Objects, specs, and constants go
- **Visual testing conventions** — component-level screenshots with `@visual` tag
- **API mocking patterns** — deterministic data via `mockApi` utility
- **Locator strategy** — role-based > data-attribute > CSS selector
- **Anti-patterns** — no `waitForTimeout`, no full-page screenshots, no fixture files

## Generated Test Structure

```
__tests__/
├── e2e/
│   ├── pages/                # Page Object classes
│   │   └── login-page.ts     #   All locators + interaction methods
│   └── login.spec.ts         # Test specs using Page Objects
└── constants/
    └── test-data.ts          # Reusable data values only
```

**Example output:**

```typescript
// __tests__/e2e/pages/login-page.ts
export class LoginPage {
  readonly page: Page;
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly loginButton: Locator;

  constructor(page: Page) {
    this.page = page;
    this.emailInput = page.getByRole('textbox', { name: 'Email' });
    this.passwordInput = page.getByRole('textbox', { name: 'Password' });
    this.loginButton = page.getByRole('button', { name: 'Login' });
  }

  async login(email: string, password: string): Promise<void> {
    await this.emailInput.waitFor({ state: 'visible' });
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.loginButton.click();
  }
}
```

```typescript
// __tests__/e2e/login.spec.ts
test.describe('User Login', () => {
  test('Valid Login @visual', async ({ page }) => {
    const loginPage = new LoginPage(page);
    await loginPage.goto();
    await loginPage.login(TEST_DATA.USER.EMAIL, TEST_DATA.USER.PASSWORD);
    await loginPage.waitForDashboard();
    await expect(loginPage.userName.locator('..')).toHaveScreenshot('dashboard-loaded.png');
  });
});
```

---

## Prerequisites

- [Claude Code](https://claude.com/claude-code) CLI installed
- [playwright-cli](https://github.com/microsoft/playwright-cli) — install globally:

```bash
npm install -g @playwright/cli@latest
```

## Installation

### Option 1: Plugin Marketplace (recommended)

```bash
# Add the marketplace
/plugin marketplace add yusuftayman/playwright-cli-agents

# Install the plugin
/plugin install playwright-cli-agents@playwright-cli-agents
```

### Option 2: Copy into your project

```bash
# Clone this repo
git clone https://github.com/yusuftayman/playwright-cli-agents.git

# Copy agents and skills into your project
cp -r playwright-cli-agents/plugins/playwright-cli-agents/agents/* your-project/.claude/agents/
cp -r playwright-cli-agents/plugins/playwright-cli-agents/skills/* your-project/.claude/skills/
```

## Usage

Once installed, Claude Code automatically delegates to these agents based on your request:

```
You: "Create a test plan for https://myapp.com/settings"
→ Claude spawns the Planner agent

You: "Generate Playwright tests for the user profile page"
→ Claude spawns the Generator agent

You: "The checkout test is failing, fix it"
→ Claude spawns the Healer agent
```

You can also invoke agents directly:

```
You: "Use the playwright-cli-planner agent to explore https://myapp.com/dashboard"
```

## Customization

### Adapt to your project

1. **Test structure** — Update paths in `SKILL.md` if your tests live somewhere other than `__tests__/e2e/`
2. **Visual testing** — Adjust `maxDiffPixels` threshold in the skill docs to match your `playwright.config.ts`
3. **API mocking** — Update the `mockApi` import paths and `API_ENDPOINTS` references to match your project

### Add new agents

Create a new `.md` file in `.claude/agents/` with this template:

```yaml
---
name: your-agent-name
description: When to use this agent...
tools: Glob, Grep, Read, Write, Bash
model: sonnet
color: purple
skills:
  - e2e
  - playwright-cli
---

Your agent instructions here...
```

## Design Decisions

| Decision | Rationale |
|----------|-----------|
| **playwright-cli over MCP** | CLI commands are more token-efficient; no large tool schemas loaded into context |
| **Page Object Model** | Separates locators from test logic; one place to update when UI changes |
| **Component-level screenshots** | More stable than full-page; failures are meaningful and diffs are readable |
| **No `waitForTimeout`** | Hard-coded delays are flaky; `waitFor` conditions are deterministic |
| **Skills preloaded via frontmatter** | Agents get domain knowledge at startup without wasting turns reading files |
| **No helpers/fixtures folders** | All logic in Page Objects keeps the architecture simple and discoverable |
| **`@visual` tag convention** | Enables `--grep @visual` to run/update only visual tests separately |

## Related Projects

- [playwright-cli](https://github.com/microsoft/playwright-cli) — Token-efficient CLI for browser automation by coding agents
- [playwright-skill](https://github.com/lackeyjb/playwright-skill) — General-purpose Playwright automation as a Claude Code skill
- [claude-code-showcase](https://github.com/ChrisWiles/claude-code-showcase) — Comprehensive Claude Code configuration example
- [everything-claude-code](https://github.com/affaan-m/everything-claude-code) — Battle-tested Claude Code configs from an Anthropic hackathon winner

## License

MIT
