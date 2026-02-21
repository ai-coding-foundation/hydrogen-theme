# AGENTS.md — Hydrogen Theme

> Cross-platform agent instructions. Read by Cursor, GitHub Copilot, OpenAI Codex, Claude Code, Google Jules, Aider, and any tool that supports the AGENTS.md standard.

---

## Project Identity

**Hydrogen Theme** is a port of Shopify Hydrogen's default template to Shopify OS 2.0, bringing modern development patterns to Shopify themes with an island architecture and hydration directives.

- **Tech stack:** Liquid, JavaScript (ES modules), Vite (rolldown-vite), Tailwind CSS 4, pnpm
- **Architecture:** Progressive enhancement via Web Components with Astro-inspired hydration strategies (`client:idle`, `client:visible`, `client:media`). Liquid templates handle server-side rendering; JavaScript islands add interactivity.
- **Build tooling:** `vite-plugin-shopify` (asset pipeline), `vite-plugin-shopify-import-maps` (module resolution), `@tailwindcss/vite` (CSS), `concurrently` (parallel dev servers)
- **Code quality:** ESLint, Prettier with `@shopify/prettier-plugin-liquid` and `prettier-plugin-tailwindcss`, Shopify Theme Check
- **CI:** GitHub Actions — ESLint + Shopify Theme Check on push/PR to `main`

---

## Commands

```bash
# Install dependencies
pnpm install

# Development
pnpm dev -- --store {your-store}   # Shopify CLI + Vite dev servers in parallel

# Build & Deploy
pnpm build                         # Vite production build
pnpm run deploy                    # Build + shopify theme push

# Quality
pnpm lint                          # ESLint
pnpm format                        # Prettier (auto-fix)
pnpm format-check                  # Prettier (check only)
```

Run `pnpm lint && pnpm format-check` before every commit.

---

## Project Structure

```
hydrogen-theme/
  frontend/                # Modern frontend code
    entrypoints/           # theme.js and theme.css — Vite entry points
    islands/               # 16 interactive Web Components (cart, product, header, etc.)
    lib/                   # revive.js (hydration engine), a11y.js, utils.js
    styles/                # CSS layers: base, components, theme, utilities
  assets/                  # Shopify theme assets (images, fonts, Vite output)
  blocks/                  # 24 reusable theme blocks
  config/                  # settings_data.json, settings_schema.json
  layout/                  # theme.liquid, password.liquid
  locales/                 # Translation files (en.default.json)
  sections/                # 23 theme sections
  snippets/                # 32 reusable Liquid snippets
  templates/               # 13 page templates (JSON + Liquid)
  .agents/                 # Hatch3r agent infrastructure (rules, agents, skills, commands)
  .github/                 # CI workflows, Copilot config, agent prompts
```

---

## Island Architecture

The theme uses an Astro-inspired island architecture for progressive enhancement. Server-rendered Liquid templates provide the base experience; JavaScript islands hydrate interactively on the client.

### Hydration Directives

| Directive        | Description                                                    |
| :--------------- | :------------------------------------------------------------- |
| `client:idle`    | Hydrate when the main thread is free (`requestIdleCallback`)   |
| `client:visible` | Hydrate when the element enters the viewport (`IntersectionObserver`) |
| `client:media`   | Hydrate when a media query matches (`matchMedia`)              |

Usage in Liquid templates:

```html
<my-component client:visible>This is an island.</my-component>
```

### Core Files

- `frontend/lib/revive.js` — Hydration engine that scans for directives and lazy-initializes islands
- `frontend/lib/a11y.js` — Accessibility utilities (focus trap, disclosure widgets)
- `frontend/lib/utils.js` — Shared helpers (`fetchConfig`, `debounce`)
- `frontend/entrypoints/theme.js` — Main entry point that initializes disclosure widgets and revives islands

### Islands

| Category | Components |
| :------- | :--------- |
| Cart     | `cart-drawer`, `cart-drawer-items`, `cart-items`, `cart-note`, `cart-remove-button` |
| Product  | `product-form`, `product-recommendations`, `variant-radios`, `variant-selects` |
| UI       | `header-drawer`, `sticky-header`, `quantity-input`, `details-modal`, `details-disclosure` |
| Other    | `localization-form`, `password-modal` |

---

## Boundaries

### Always do

- Run `pnpm lint && pnpm format-check` before committing
- Use hydration directives (`client:idle`, `client:visible`, `client:media`) for interactive components
- Follow Shopify OS 2.0 theme conventions (sections, blocks, JSON templates)
- Keep islands small and focused on a single interactive concern
- Use `fetchConfig()` from `frontend/lib/utils.js` for Shopify AJAX API calls
- Follow the existing code style (single quotes, no semicolons, ES modules)
- Keep PRs focused: one issue per PR

### Ask first

- Adding new npm dependencies
- Creating new sections or blocks
- Changing `config/settings_schema.json` or `config/settings_data.json`
- Architectural decisions that affect the hydration system
- Adding new hydration directives

### Never do

- Commit secrets, API keys, or `.env` files
- Force push to `main`
- Bypass the island hydration system for interactive components (use Web Components + directives)
- Use inline `<script>` tags for functionality that should be an island
- Use `any` types without justification (in JSDoc annotations)
- Expand scope beyond acceptance criteria
- Modify `assets/.vite/*` output files directly (managed by the build)

---

## Environment Variables

The `.env` file is gitignored. Create it manually with required secrets. No `.env.example` currently exists.

For Cursor MCP: `.cursor/mcp.json` is gitignored. Copy `.cursor/mcp.json.example` if available, or configure MCP servers manually.

---

## Cursor Agent Setup

Cursor users get additional specialized guidance via the hatch3r agent infrastructure:

- **Rules** (`/.agents/rules/`): 11 rules covering component conventions, tooling hierarchy, error handling, API design, migrations, feature flags, performance budgets, code standards, testing, dependency management, observability.
- **Skills** (`/.agents/skills/`): 19 workflow skills including Shopify expert, bug fix, feature implementation, code/logical/visual refactor, QA validation, PR creation, release, incident response, and more.
- **Agents** (`/.agents/agents/`): 10 specialized agents for code review, test writing, security auditing, lint fixing, documentation, accessibility auditing, CI watching, performance profiling, dependency auditing, and per-sub-issue implementation delegation.
- **Commands** (`/.agents/commands/`): 7 commands for board management (`board-init`, `board-fill`, `board-pickup`), healthcheck, release, and dependency audit.

---

## Additional Skill assignment

Always use these skills when involving these specific topics:
- **nano-banana-pro** (`/.agents/skills/nano-banana-pro`): when images or assets are needed on the fly.
- **seo-audit** (`/.agents/skills/seo-audit`): when SEO is part of the task
- **ui-ux-pro-max** (`/.agents/skills/ui-ux-pro-max`): When user interface, design or user experience topics are part of the task.

---

## Board Management

This project uses three commands and an auto-maintained GitHub issue to manage the project board:

| Primitive           | What it does                                                                                                                                                                           | When to use                                                                                  |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| **`board-fill`**    | Creates epics/issues from `todo.md`, reorganizes the board (dependencies, grouping, readiness), syncs Projects v2 status, and updates the Board Overview super-epic.                   | When new work items need to be added, or existing issues need dependency analysis and triage. |
| **`board-pickup`**  | Presents available work, auto-picks the next best issue when none is specified, performs collision detection, creates a branch, and delegates to the issue-workflow skill.              | When starting development on an issue.                                                       |
| **`board-refresh`** | Audits Projects v2 board status against issue labels, reconciles drift, and regenerates the Board Overview. Does not modify issues — only board status and the overview.               | When the board looks out of sync, after manual label changes, or to get a fresh overview.    |
| **Board Overview**  | A single GitHub issue (`meta:board-overview` label) listing all epics, issues, implementation order, parallel lanes, and live statuses. Auto-maintained by all three commands.         | Read it for a quick overview of the entire board state.                                      |

**Typical cycle:** Add items to `todo.md` --> run `board-fill` --> review Board Overview --> run `board-pickup` (or let it auto-pick) --> develop --> open PR --> Board Overview refreshes automatically. Run `board-refresh` at any time to reconcile drift.

---

## Working with Issues

Each issue type has a dedicated skill in `/.agents/skills/`:

| Issue Type        | Skill                                              |
| ----------------- | -------------------------------------------------- |
| Bug report        | `/.agents/skills/hatch3r-bug-fix/SKILL.md`         |
| Feature request   | `/.agents/skills/hatch3r-feature/SKILL.md`         |
| Code refactor     | `/.agents/skills/hatch3r-refactor/SKILL.md`        |
| Logical refactor  | `/.agents/skills/hatch3r-logical-refactor/SKILL.md`|
| Visual refactor   | `/.agents/skills/hatch3r-visual-refactor/SKILL.md` |
| QA E2E validation | `/.agents/skills/hatch3r-qa-validation/SKILL.md`   |

**Workflow:** Read issue --> load issue-type skill --> plan --> implement --> test --> open PR. Full details in the `hatch3r-issue-workflow` skill.

**Epic sub-agent delegation:** When picking up an epic with multiple sub-issues, `board-pickup` delegates each sub-issue to a dedicated `implementer` sub-agent (`/.agents/agents/hatch3r-implementer.md`). Sub-issues at the same dependency level run in parallel (max 4 concurrent). The parent orchestrator handles branch creation, git operations, commits, PRs, and board status.

---

## Tooling Hierarchy

Agents follow a strict tooling hierarchy for operations and knowledge acquisition.

### GitHub: MCP-First

Always prefer GitHub MCP tools over `gh` CLI. The MCP server provides typed tools for issues, pull requests, projects, and more. Fall back to `gh` CLI only for operations outside the MCP catalog (e.g., `gh run` for Actions workflows).

### Knowledge Augmentation Priority

1. **Codebase exploration** (Grep, SemanticSearch, explore sub-agents) — ground truth for current implementation
2. **Context7 MCP** — use `resolve-library-id` then `query-docs` to pull up-to-date library documentation (Shopify Liquid, Tailwind CSS, Vite, etc.)
3. **Web research** — current events, security advisories, breaking changes, best practices

---

## Git Workflow

- Branch from `main`: `{type}/{short-description}` (e.g., `feat/product-gallery`, `fix/cart-drawer-overflow`)
- Squash merge. Delete branches after merge. No force push to `main`.
- PR title: `{type}: {short description} (#issue)`
- Run `pnpm lint && pnpm format-check` before every commit.

---

## Error Recovery

- After 3 failed attempts at the same fix, stop and escalate with what was tried and why it failed.
- On rate limits, back off exponentially (2s -> 4s -> 8s -> 16s).
- Report what succeeded, what failed, and propose recovery options.

## Context Management

- Use targeted searches over reading entire large files.
- For epics with 3+ sub-issues, delegate to implementer sub-agents.
- If context feels degraded after ~25 turns, start a fresh chat with a progress summary.
