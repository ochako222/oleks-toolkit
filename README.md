# oleksandr-toolkit

A Claude Code plugin with reusable agents, commands, skills, and templates for full-stack React + Flask development.

## What's included

| Category | Count | Description |
|----------|-------|-------------|
| Agents | 6 | Specialized subagents for code review, frontend, and testing |
| Commands | 5 | Flow orchestrators including `/synchronize` for plugin sync |
| Skills | 15 | Reusable workflows for frontend, architecture, and Playwright |
| Templates | 15 | Code pattern references for consistent generation |

## Installation

```bash
claude install ochako222/oleks-toolkit
```

## Agents

| Agent | Purpose |
|-------|---------|
| `architecture-code-reviewer` | Senior code reviewer — runs Biome, checks architecture |
| `architecture-consistency-guard` | CI-style guard for Command→Skill→Template wiring gaps |
| `frontend-mobile-responsive` | rem-first responsive design, mobile-first patterns |
| `playwright-test-generator` | Generates Playwright tests using MCP browser tools |
| `playwright-test-healer` | Debugs and fixes failing Playwright tests |
| `playwright-test-planner` | Creates comprehensive Playwright test plans |

## Commands

| Command | Purpose |
|---------|---------|
| `/architecture-scanner` | Scans `.claude/` hierarchy for consistency issues |
| `/architecture-synchronizer` | Enforces Command→Skill→Template hierarchy |
| `/claude-skill-builder` | Interactive Q&A entity creator |
| `/codder` | Conversational coding assistant for frontend/backend |
| `/synchronize` | Bidirectional sync between this plugin repo and your project's `.claude/` |

## Skills

### Architecture
- `architecture-agent-builder` — Build and audit Claude Code agents
- `architecture-command-builder` — Build and audit custom commands
- `architecture-skill-builder` — Create and restructure skills
- `architecture-template-builder` — Build and audit templates

### Frontend
- `frontend-api-service-builder` — API service classes, useApiCall hook, ApiError
- `frontend-page-architector` — Full flow architecture (list → details → cards)
- `frontend-pattern-advisor` — HOC / Render Props decision framework
- `frontend-performance-optimizer` — React Compiler-aware memoization audit
- `frontend-portal-builder` — Portal-based modals, tooltips, toasts
- `frontend-scss-writer` — Minimalist SCSS following project conventions
- `frontend-state-manager` — React Context, Zustand, Redux Toolkit
- `frontend-vitest-rtl` — Vitest + RTL component tests

### Playwright E2E
- `playwright-pom-generator` — Page Object Model classes
- `playwright-test-healer` — Fix failing E2E tests
- `playwright-test-writer` — Write E2E test specs

## Templates

Code pattern references used by skills during generation:

- `api-axios-config` — Axios setup with ApiError class
- `api-service-class` — Service class composition pattern
- `frontend-flow-card-component` — Editable card component
- `frontend-flow-details-page` — Entity details page with tabs
- `frontend-flow-entity-slice` — Redux slice for entity management
- `frontend-flow-list-page` — List page with table and filters
- `frontend-flow-view-component` — Read-only view component
- `frontend-hoc-pattern` — Higher-order component pattern
- `frontend-performance-memo-patterns` — useMemo/useCallback/memo patterns
- `frontend-portal-pattern` — React portal infrastructure
- `frontend-react-context` — React Context with reducer
- `frontend-redux-slice` — Redux Toolkit slice
- `frontend-render-props-pattern` — Render props pattern
- `frontend-zustand-store` — Zustand store pattern
- `use-api-call-hook` — useApiCall hook implementation

## `/synchronize` command

The `/synchronize` command keeps this plugin in sync with your project:

- **Pull from plugin** → get latest agents, commands, skills, templates
- **Push to plugin** → contribute your project's improvements back
- **Bidirectional** → pull plugin-only, push local-only, decide on diverged
- **Pick specific** → cherry-pick individual entities

Run it any time to see what's new or what your project has that the plugin doesn't.

## License

MIT
