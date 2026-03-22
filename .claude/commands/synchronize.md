---
name: synchronize
description: >
  Synchronize Claude Code entities between the oleksandr-toolkit GitHub plugin
  (ochako222/oleks-toolkit) and the current project's .claude/ folder.
  Bidirectional comparison: pull new/updated entities from the plugin, push
  project improvements back to the plugin. Use when: "sync", "synchronize",
  "update from plugin", "push to plugin".
disable-model-invocation: true
---

# Synchronize

Bidirectional sync between `ochako222/oleks-toolkit` (the plugin) and the current project's `.claude/` folder.

## Flow

### Step 1: Read Context

Read `CLAUDE.md` at the project root to understand the project structure.

### Step 2: Inventory Plugin Repo

Use `mcp__github__get_file_contents` to read each entity from the plugin repo (`ochako222/oleks-toolkit`, branch `main`).

**Entity paths to read:**

Agents:
- `.claude/agents/architecture-code-reviewer.md`
- `.claude/agents/architecture-consistency-guard.md`
- `.claude/agents/frontend-mobile-responsive.md`
- `.claude/agents/playwright-test-generator.md`
- `.claude/agents/playwright-test-healer.md`
- `.claude/agents/playwright-test-planner.md`

Commands:
- `.claude/commands/architecture-scanner.md`
- `.claude/commands/architecture-synchronizer.md`
- `.claude/commands/claude-skill-builder.md`
- `.claude/commands/codder.md`
- `.claude/commands/synchronize.md`

Skills:
- `.claude/skills/architecture-agent-builder/SKILL.md`
- `.claude/skills/architecture-command-builder/SKILL.md`
- `.claude/skills/architecture-skill-builder/SKILL.md`
- `.claude/skills/architecture-template-builder/SKILL.md`
- `.claude/skills/frontend-api-service-builder/SKILL.md`
- `.claude/skills/frontend-page-architector/SKILL.md`
- `.claude/skills/frontend-pattern-advisor/SKILL.md`
- `.claude/skills/frontend-performance-optimizer/SKILL.md`
- `.claude/skills/frontend-portal-builder/SKILL.md`
- `.claude/skills/frontend-scss-writer/SKILL.md`
- `.claude/skills/frontend-state-manager/SKILL.md`
- `.claude/skills/frontend-vitest-rtl/SKILL.md`
- `.claude/skills/playwright-pom-generator/SKILL.md`
- `.claude/skills/playwright-test-healer/SKILL.md`
- `.claude/skills/playwright-test-writer/SKILL.md`

Templates:
- `.claude/templates/api-axios-config.md`
- `.claude/templates/api-service-class.md`
- `.claude/templates/frontend-flow-card-component.md`
- `.claude/templates/frontend-flow-details-page.md`
- `.claude/templates/frontend-flow-entity-slice.md`
- `.claude/templates/frontend-flow-list-page.md`
- `.claude/templates/frontend-flow-view-component.md`
- `.claude/templates/frontend-hoc-pattern.md`
- `.claude/templates/frontend-performance-memo-patterns.md`
- `.claude/templates/frontend-portal-pattern.md`
- `.claude/templates/frontend-react-context.md`
- `.claude/templates/frontend-redux-slice.md`
- `.claude/templates/frontend-render-props-pattern.md`
- `.claude/templates/frontend-zustand-store.md`
- `.claude/templates/use-api-call-hook.md`

Store all plugin file contents keyed by their path.

### Step 3: Inventory Local `.claude/` Folder

Use Glob to discover local entities:

```
.claude/agents/*.md
.claude/commands/*.md
.claude/skills/*/SKILL.md
.claude/templates/*.md
```

For each discovered file, read its content with the Read tool. Store all local file contents keyed by their relative path.

### Step 4: Compare

For each entity path, determine sync status:

| Status | Condition |
|--------|-----------|
| `IN_SYNC` | File exists in both and content is identical |
| `DIVERGED` | File exists in both but content differs |
| `PLUGIN_ONLY` | File exists in plugin repo but NOT locally |
| `LOCAL_ONLY` | File exists locally but NOT in plugin repo |

Build the full comparison table.

### Step 5: Enter Plan Mode

Call `EnterPlanMode` before presenting findings.

### Step 6: Present Sync Report

Output a structured table grouped by category:

---

## Sync Report — oleksandr-toolkit ↔ Local `.claude/`

> **Plugin repo**: `ochako222/oleks-toolkit`
> **Local project**: Current working directory

### Agents

| Entity | Status |
|--------|--------|
| `architecture-code-reviewer` | IN_SYNC / DIVERGED / PLUGIN_ONLY / LOCAL_ONLY |
| ... | ... |

### Commands

| Entity | Status |
|--------|--------|
| `synchronize` | IN_SYNC / DIVERGED / PLUGIN_ONLY / LOCAL_ONLY |
| ... | ... |

### Skills

| Entity | Status |
|--------|--------|
| `frontend-vitest-rtl` | IN_SYNC / DIVERGED / PLUGIN_ONLY / LOCAL_ONLY |
| ... | ... |

### Templates

| Entity | Status |
|--------|--------|
| `api-axios-config` | IN_SYNC / DIVERGED / PLUGIN_ONLY / LOCAL_ONLY |
| ... | ... |

---

**Summary**: N in sync, N diverged, N plugin-only, N local-only

### Step 7: Ask for Action

Ask the user using `AskUserQuestion`:

> "What would you like to do?"

Options:
- **Pull all from plugin** — overwrite all local files with plugin versions (PLUGIN_ONLY + DIVERGED entities)
- **Push all to plugin** — push all local files to plugin repo (LOCAL_ONLY + DIVERGED entities)
- **Bidirectional** — pull PLUGIN_ONLY, push LOCAL_ONLY (ask about DIVERGED individually)
- **Pick specific** — select individual entities to sync
- **Done** — exit without changes

### Step 8: Apply Changes

**For PULL operations** (plugin → local):

Use the Write tool to write/overwrite local files with plugin content from Step 2.

**For PUSH operations** (local → plugin):

Use `mcp__github__push_files` to push files to `ochako222/oleks-toolkit` on branch `main`.

Format each file as `{ "path": "<entity-path>", "content": "<content>" }`.

Push in batches of up to 10 files per commit to avoid payload limits.

**For DIVERGED entities in bidirectional mode**, ask the user per entity:

> "Entity `<name>` differs. Which version to keep?"
> - Plugin version → pull (overwrite local)
> - Local version → push (overwrite plugin)
> - Skip → leave both as-is

### Step 9: Confirm

After all changes are applied, output:

> "Sync complete. Pulled: N entities. Pushed: N entities. Skipped: N entities."

## Rules

- **Always enter plan mode** before presenting the report (Step 5)
- **Never apply changes without user approval** — the report is always shown first
- **Skills are in subdirectories** — path is `.claude/skills/<name>/SKILL.md` in both local and plugin
- **Identical content = IN_SYNC** — compare full file text, not just frontmatter
- **DIVERGED entities in bidirectional mode** — always ask the user which version wins
- **GitHub push uses branch `main`** — always push to `ochako222/oleks-toolkit` on branch `main`
- **Read CLAUDE.md first** — understand the project before comparing entities
- **Batch GitHub pushes** — max 10 files per `mcp__github__push_files` call
