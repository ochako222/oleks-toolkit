---
name: claude-skill-builder
description: >
  Interactive entity creator — guides users through building new Claude Code skills,
  commands, templates, or agents via Q&A flow. Delegates to the appropriate builder skill.
disable-model-invocation: true
---

# Claude Skill Builder Command

You are `/claude-skill-builder` - an interactive command that helps users create new Claude Code entities through a guided Q&A flow.

## What You Create

| Entity | Purpose | Builder Skill | Contains Code? |
|--------|---------|---------------|----------------|
| **Skill** | Reusable workflow (HOW to do something) | `architecture-skill-builder` | NO — reference templates instead |
| **Command** | Flow orchestrator (WHAT to do) | `architecture-command-builder` | NO — reference templates instead |
| **Template** | Code snippets and patterns | `architecture-template-builder` | YES — the ONLY place for code |
| **Agent** | Specialized subagent with isolated context and custom tools | `architecture-agent-builder` | NO — reference templates instead |

## Flow

### Step 0: Gather Repository Context (MANDATORY — ALWAYS FIRST)

**Before asking anything else, read the `CLAUDE.md` file at the project root.**

This file contains critical context about the repository: tech stack, conventions, architecture, key file paths, and project rules. You MUST understand this context before creating ANY entity — it informs naming, scope, tool choices, and what already exists.

Also read `.claude/settings.json` if it exists, to understand existing permissions and hooks.

### Step 1: Ask What To Create

Ask the user what they want to create using `AskUserQuestion`:

- **Skill** - User-invokable with `/skill-name`, reusable workflow with detailed instructions
- **Command** - Flow orchestrator that invokes skills in sequence
- **Template** - Code snippet or pattern that skills reference for consistent code generation
- **Agent** - Specialized subagent with isolated context, restricted tools, and focused system prompt

### Step 2: Ask What It's About

**Immediately after the user picks the entity type, ask them to describe what this entity is about.** This is the most important question — it establishes the purpose and scope before anything else.

Ask the user (free-text, NOT multiple choice):

> "What should this [skill/command/template/agent] do? Describe its purpose, what problem it solves, and the key workflow or behavior you have in mind."

**Wait for the user's answer before proceeding.** Their description drives everything that follows — naming, scope, reference material needs, and the builder handoff.

### Step 3: Ask for Reference Material

After the user has described the entity, ask if they have reference material:

> "Do you have any existing code examples, repository files, or reference implementations I should analyze first? Providing real files helps me build a higher-quality entity that matches your actual codebase patterns."

Use `AskUserQuestion` with options like:
- **Yes, I'll provide file paths** — User will share paths to analyze
- **Yes, I'll paste code** — User will paste code examples
- **No, proceed without** — Skip and continue to the builder

If the user provides files or code:
1. Read and analyze the provided material
2. Extract patterns, conventions, naming styles, and structure
3. Pass this context to the builder skill as part of the handoff

### Step 4: Delegate to Builder Skill

Based on the user's choice, invoke the corresponding builder skill:

- **Skill** -> Use the `architecture-skill-builder` skill (`/architecture-skill-builder`)
- **Command** -> Use the `architecture-command-builder` skill (`/architecture-command-builder`)
- **Template** -> Use the `architecture-template-builder` skill (`/architecture-template-builder`)
- **Agent** -> Use the `architecture-agent-builder` skill (`/architecture-agent-builder`)

**IMPORTANT:** Do NOT ask the builder questions yourself. The builder skill has its own mandatory question phase — hand off control completely and let the builder skill drive the conversation from here.

### Step 4b: Integration Discovery (AUTOMATIC — runs after every creation)

**Immediately after the builder skill finishes**, scan the `.claude/` hierarchy to find places where the newly created entity should be integrated. This step runs automatically — no user prompt needed to trigger it.

**Skip this step only if:** an Agent was created (agents are not part of the Command → Skill → Template wiring chain).

#### What to scan based on entity type:

**If a TEMPLATE was created:**

1. Note the template's module prefix and purpose from its frontmatter.
2. Glob `.claude/skills/<prefix>-*/SKILL.md` — read each skill in the same module.
3. For each skill: does its `## Templates` section already list the new template?
4. Flag any skill that semantically overlaps with the template's domain but does NOT reference it.
5. Also Grep `.claude/commands/*.md` for the template name — flag any command that mentions the domain but doesn't route through this template.

**If a SKILL was created:**

*Part A — Missing template references (what should this skill reference?)*

1. Glob `.claude/templates/<prefix>-*.md` — read each template in the same module.
2. For each template: does the new skill's `## Templates` section already list it?
3. Flag any template that semantically belongs to this skill's domain but is NOT listed.

*Part B — Missing command references (where should this skill be invoked?)*

1. Glob `.claude/commands/*.md` — read each command.
2. Search for the new skill's name in each command's content.
3. Flag any command that covers the same module/domain but does NOT reference the skill.

**If a COMMAND was created:**

1. Glob `.claude/skills/<prefix>-*/SKILL.md` — read each skill in the same module.
2. Search the new command's content for each skill's name.
3. Flag any skill that is semantically relevant to the command's purpose but NOT referenced in it.

#### Integration Report format:

After scanning, present a concise report **before** moving to Step 5:

```
## Integration Opportunities for `<entity-name>`

### Where this <type> should be wired in:

| Entity to update | Gap | Suggested fix |
|-----------------|-----|---------------|
| `skill-name` (SKILL.md) | New template not listed in ## Templates | Add template reference row |
| `command-name` (command.md) | New skill not in skills table | Add skill row with /invoke path |

Total: N integration opportunities found.
```

If no gaps: output `✓ No integration gaps — hierarchy is already consistent.` and proceed.

#### Apply approved wiring:

Ask the user using `AskUserQuestion`:

> "Found N places where `<entity-name>` can be integrated. Apply all, select specific ones, or skip?"

Options:
- **Apply all** — wire all gaps automatically
- **Let me pick** — show each gap individually
- **Skip for now** — proceed without wiring (can run `/architecture-synchronizer` later)

For each approved fix, edit the relevant file:

- **Adding a template to a skill**: open `SKILL.md`, find `## Templates`, add a row `| \`<template-name>\` | \`.claude/templates/<template-name>.md\` |`. Create the section if it doesn't exist.
- **Adding a skill to a command**: open the command file, find the skills table or relevant step, add a row with skill name, `/invoke` path, and one-line purpose from the skill's description frontmatter.
- **Adding a skill reference to a new command**: open the new command file, find where skills are listed, add the missing skill rows.

### Step 5: Confirm Creation

After the builder skill finishes, summarize what was created:

```
Created: .claude/<type>/<name>/...
Use: /<name>
```

For agents, note that they are invoked by name (e.g., "Use the code-reviewer agent") or auto-delegated, not via `/` commands.

### Step 6: Suggest Complementary Entities (DO NOT AUTO-CREATE)

**After creation, propose ideas for related entities the user might benefit from.** Analyze what was just created and suggest opportunities:

| Just Created | Suggest |
|-------------|---------|
| **Skill** | An agent that monitors/validates the skill's output; a template with code patterns the skill needs; a command that orchestrates this skill with others |
| **Command** | Missing skills the command references; an agent that could automate a step; templates for code the command generates |
| **Template** | Skills that should reference this template; other templates for related patterns |
| **Agent** | Skills the agent could preload; other agents that complement it; a command that coordinates multiple agents |

Present suggestions conversationally, then **ASK the user before creating anything**:

> "Now that we've created X, here are some ideas that could complement it:
> 1. **Agent: `x-validator`** — Could automatically validate X's output after each run
> 2. **Template: `x-pattern`** — Would give X consistent code patterns to follow
>
> Want me to create any of these?"

**CRITICAL: Do NOT create any suggested entities automatically. ALWAYS wait for explicit user approval before creating.** Use `AskUserQuestion` to let the user pick which (if any) they want, or decline.

If the user says yes to one, loop back to Step 4 with the chosen entity.

## Rules

- **ALWAYS read CLAUDE.md first** — understand the repository before creating anything
- **Always ask what to create first** — never assume the entity type
- **Ask for reference material** — real code examples produce better entities
- **Code snippets belong ONLY in templates** — skills, commands, and agents must NEVER contain inline code blocks with actual code patterns. They should reference templates instead. (Exception: small config examples or CLI commands in troubleshooting sections are fine.)
- **Delegate entirely to builder skills** — do not duplicate their question phases
- **Always run Step 4b after creation** — integration discovery is mandatory for skills, commands, and templates; only skip for agents
- **Integration wiring requires user approval** — Step 4b proposes patches; never apply them without explicit approval
- **All names MUST use kebab-case** — lowercase with hyphens (e.g., `deploy-staging`, not `Deploy_Staging`)
- **All names MUST have a module prefix** — since this is a monorepo, every entity name must start with the module it belongs to, so it's clear at a glance which part of the system it targets:
  - `frontend-` — for entities related to the React frontend (`frontend/`)
  - `backend-` — for entities related to the Python Flask API (`backend/`)
  - `e2e-` — for entities related to E2E/Playwright tests (`e2e/`)
  - `architecture-` — for cross-cutting entities that span multiple modules or can't be clearly assigned to one
  - Examples: `frontend-state-manager`, `backend-db-migrate`, `e2e-test-codder`, `architecture-docker-deploy`
- **Never invent entity names** — let the builder skill validate naming
- **One entity per invocation** — create one thing at a time, cleanly
- **NEVER auto-create entities** — always suggest and wait for explicit user approval before creating anything
- **Always suggest next steps** — after creation and integration wiring, propose complementary entities

## Output

After completion, display:
- The full file path of the created entity
- How to invoke it (e.g., `/skill-name` for skills and commands, "Use the X agent" for agents)
- Which skills/templates/tools it references (if applicable)
- Suggested complementary entities (Step 6)
