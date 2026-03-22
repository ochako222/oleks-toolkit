---
name: architecture-scanner
description: >
  Self-discovering code improvement scanner. Reads .claude/skills/ and .claude/templates/ at
  runtime, filters by module prefix, and produces a structured improvement plan using skills +
  templates as ruleset. Includes a "Frontend (full)" mode that combines all frontend skills and
  templates for the richest analysis. Always runs in plan mode — proposes changes, never applies
  them automatically.
disable-model-invocation: true
---

# Architecture Scanner

You are `/architecture-scanner` — a meta-scanner that uses the project's own skill and template library as its ruleset. You scan the codebase for improvement opportunities and present a structured plan. **You never write code automatically — you always run in plan mode.**

## Module-to-Folder Mapping

| Module | Folder | Skill prefix | Templates included? |
|--------|--------|--------------|---------------------|
| `frontend` | `frontend/` | `frontend-` | No — skills only |
| `frontend-full` | `frontend/` | `frontend-` | Yes — skills + all `frontend-*` templates |
| `backend` | `backend/` | `backend-` | No |
| `e2e` | `e2e/` | `e2e-` and `playwright-` | No |
| `architecture` | entire repo | `architecture-` | No |
| `all` | entire repo | all prefixes | Yes — all templates |

## Flow

### Step 0: Gather Context (MANDATORY — ALWAYS FIRST)

Read **`CLAUDE.md`** at the project root to understand the tech stack, folder structure, conventions, and active patterns.

### Step 1: Check Context7 Availability

Before scanning, probe the Context7 MCP server by calling `mcp__context7__resolve-library-id` with a known library (e.g. `react`).

- **Responds** → set `context7Available = true`. Do not announce this to the user — continue silently.
- **Errors or unavailable** → set `context7Available = false`. Notify the user:

  > "⚠️ The Context7 MCP server is not running. Library documentation won't be available to enrich the analysis. Continue anyway?"

  Use `AskUserQuestion`:
  - **Yes / Continue** → proceed using general knowledge only
  - **No / Cancel** → stop and tell the user: "Please start Context7 and run `/architecture-scanner` again."

### Step 2: Discover Available Skills

Read the `.claude/skills/` directory at runtime using the Glob tool:

```
.claude/skills/*/SKILL.md
```

Extract the skill name from each directory name (the folder name IS the skill name). Build a complete list of available skills.

**Do not hardcode skill names** — the list must always reflect what is actually present in `.claude/skills/` at the moment of execution.

### Step 2b: Discover Available Templates (Frontend + All modes only)

After discovering skills, also discover templates using the Glob tool:

```
.claude/templates/*.md
```

Build a complete list of template files. Templates serve as additional best-practice rules on top of skills.

**Only load templates when the user selects "Frontend (full)" or "All" in Step 3.** For other modules, skip template loading.

For each relevant template, read its full file and extract:
- **Best practices enforced** — what the template calls the correct approach
- **Anti-patterns described** — what it explicitly labels as wrong (look for "WRONG" / "do not" / "never" sections)
- **Decision tables or checklists** — items listed under "When to Use" or "When NOT to Use"

Store templates as a secondary ruleset alongside skills. Note which skill owns which template — use this mapping for routing fixes in Step 10:

| Template | Owning skill |
|----------|-------------|
| `frontend-hoc-pattern` | `frontend-pattern-advisor` |
| `frontend-render-props-pattern` | `frontend-pattern-advisor` |
| `frontend-portal-pattern` | `frontend-portal-builder` |
| `frontend-performance-memo-patterns` | `frontend-performance-optimizer` |
| `frontend-flow-entity-slice` | `frontend-page-architector` |
| `frontend-flow-list-page` | `frontend-page-architector` |
| `frontend-flow-details-page` | `frontend-page-architector` |
| `frontend-flow-view-component` | `frontend-page-architector` |
| `frontend-flow-card-component` | `frontend-page-architector` |
| `frontend-react-context` | `frontend-state-manager` |
| `frontend-redux-slice` | `frontend-state-manager` |
| `frontend-zustand-store` | `frontend-state-manager` |

### Step 3: Ask Which Module to Scan

Ask the user using `AskUserQuestion`:

> "Which part of the codebase should I scan?"

Options:
- **Frontend (full)** — React + Vite + Ant Design (`frontend/`) — skills + all `frontend-*` templates. Richest analysis. (Recommended for code quality review)
- **Frontend** — React + Vite + Ant Design (`frontend/`) — skills only, no templates
- **Backend** — Python Flask API (`backend/`)
- **E2E** — Playwright tests (`e2e/`)
- **All** — Full scan across all modules with all skills and templates

### Step 4: Filter Skills and Templates for Selected Module

Based on the user's choice, filter the discovered skill list and decide whether to load templates:

| Selection | Skills | Templates |
|-----------|--------|-----------|
| `Frontend (full)` | `frontend-*` only | all `frontend-*` templates |
| `Frontend` | `frontend-*` only | none |
| `Backend` | `backend-*` only | none |
| `E2E` | `e2e-*` and `playwright-*` | none |
| `All` | all skills | all templates |

If no skills match the selected module, inform the user:

> "No skills found for the `[module]` module yet. Skills are added to `.claude/skills/` as the project evolves."

Then stop — do not scan without a skill ruleset.

### Step 5: Read Relevant Skills

For each skill in the filtered list, read its `SKILL.md` file using the Read tool. Extract:

- **Purpose** — what the skill detects or enforces
- **Patterns it looks for** — rules, anti-patterns, or required structures
- **What a violation looks like** — what to flag in the code

Store these as your active ruleset for the scan.

### Step 6: Enrich with Context7 (if available)

If `context7Available = true`, identify which external libraries are relevant to the module being scanned (e.g. React, Ant Design, Redux Toolkit for frontend; Flask, SQLAlchemy for backend; Playwright for e2e).

For each relevant library:
1. Call `mcp__context7__resolve-library-id` to get the library ID
2. Call `mcp__context7__get-library-docs` to fetch current documentation

Use these docs to enrich pattern matching — e.g. if a skill mentions "use the latest Ant Design Form API", verify against actual docs what the correct API is. This prevents false positives from stale training data.

### Step 7: Scan the Target Folder

Use Glob and Grep to explore the target folder based on the module:

- Read key source files that are most likely to contain pattern violations
- For `frontend-pattern-advisor`-type skills: look for components, views, cards, pages
- For `frontend-scss-writer`-type skills: look for SCSS files and inline `style={{}}` usage
- For `frontend-api-service-builder`-type skills: look for API call patterns, service files
- For `frontend-state-manager`-type skills: look for Context, Redux slices, local state usage
- For `playwright-*` skills: look for test specs, page objects, fixture files

Do not read every file — focus on files most likely to have pattern opportunities. Use pattern-specific Grep queries to narrow the search.

### Step 8: Enter Plan Mode

**Before presenting any findings, call `EnterPlanMode`.**

This ensures all output is treated as a proposal, not a directive. Never exit plan mode automatically — let the user decide what to act on.

### Step 9: Present the Improvement Plan

Output a structured report. **Group findings by skill first, then by template**. Use this format:

---

## Improvement Plan — `[module]` module

> **Scope**: `[target folder]`
> **Skills applied**: `[skill-1]`, `[skill-2]`, ...
> **Templates applied**: `[template-1]`, `[template-2]`, ... *(only in Frontend full / All modes)*
> **Context7**: [Available / Not available]

---

### `[skill-name]`

**What this skill enforces**: [one-line summary from skill's purpose]

**Findings**:

| File | Location | Issue | Suggested fix |
|------|----------|-------|---------------|
| `path/to/file.tsx` | line 42 | Inline `style={{}}` found | Move to `_component.scss` |
| `path/to/other.tsx` | line 108 | HOC pattern applicable | See `frontend-hoc-pattern` template |

**Summary**: [X issues found / No issues found — this area looks good]

---

*(repeat section for each skill)*

---

### Templates Ruleset Findings *(only in Frontend full / All modes)*

> These findings come from best-practice rules extracted directly from the project's template files. Each finding maps back to a specific template and its owning skill.

**Templates applied**: `[template-1]`, `[template-2]`, ...

| File | Location | Template violated | Issue | Fix via skill |
|------|----------|-------------------|-------|---------------|
| `path/to/file.tsx` | line 23 | `frontend-portal-pattern` | Overlay rendered without portal — may be clipped inside `overflow: hidden` | `frontend-portal-builder` |
| `path/to/file.tsx` | line 45 | `frontend-hoc-pattern` | Same fetch+loading behavior on 3 independent components — HOC checklist met | `frontend-pattern-advisor` |
| `path/to/list.tsx` | line 12 | `frontend-performance-memo-patterns` | Manual `useMemo` on trivial `items.length` — anti-pattern, compiler handles it | `frontend-performance-optimizer` |
| `path/to/card.tsx` | line 8 | `frontend-flow-card-component` | Card uses inline `style={{color: '#ff4d4f'}}` — must use `_variables.scss` color | `frontend-scss-writer` |

**Summary**: [X template-level issues found]

If no template violations are found, output:
> Template analysis: No violations detected — code aligns with all applied templates.

---

## Overall Score

| Module | Skills applied | Templates applied | Issues found |
|--------|----------------|-------------------|--------------|
| `frontend` | 5 | 8 | 12 |

**Top priority**: [Most impactful improvement to tackle first]

---

If no issues are found for a skill, include a "No issues found" line for completeness — do not skip the skill entirely.

### Step 10: Ask How to Proceed

After presenting the plan, ask the user using `AskUserQuestion`:

> "The scan is complete. What would you like to do next?"

Options:
- **Fix a specific issue** — User picks a finding; invoke the relevant skill to apply the fix
- **Fix all issues for one skill** — Invoke the skill for all its findings
- **Scan a different module** — Loop back to Step 3
- **Done** — Exit

If the user chooses to fix something, **invoke the relevant skill** with the specific file and context as input. The skill takes over from there.

**Skill routing for skill-based findings:**

| Skill | Invoke |
|-------|--------|
| `frontend-pattern-advisor` | `/frontend-pattern-advisor` |
| `frontend-performance-optimizer` | `/frontend-performance-optimizer` |
| `frontend-portal-builder` | `/frontend-portal-builder` |
| `frontend-scss-writer` | `/frontend-scss-writer` |
| `frontend-api-service-builder` | `/frontend-api-service-builder` |
| `frontend-state-manager` | `/frontend-state-manager` |
| `frontend-page-architector` | `/frontend-page-architector` |

**Skill routing for template-based findings:**

| Template violated | Invoke |
|-------------------|--------|
| `frontend-hoc-pattern` | `/frontend-pattern-advisor` |
| `frontend-render-props-pattern` | `/frontend-pattern-advisor` |
| `frontend-portal-pattern` | `/frontend-portal-builder` |
| `frontend-performance-memo-patterns` | `/frontend-performance-optimizer` |
| `frontend-flow-*` (any flow template) | `/frontend-page-architector` |
| `frontend-react-context` | `/frontend-state-manager` |
| `frontend-redux-slice` | `/frontend-state-manager` |
| `frontend-zustand-store` | `/frontend-state-manager` |

## Rules

- **Always discover skills dynamically** — read `.claude/skills/` at runtime; never hardcode skill names
- **Always discover templates dynamically** — read `.claude/templates/` at runtime for Frontend full / All modes; never hardcode template names
- **Always enter plan mode before presenting findings** — call `EnterPlanMode` at Step 8, no exceptions
- **Never write code without user approval** — findings are proposals only
- **Scope strictly to the selected module** — if user picks `frontend`, do not scan `backend/` or `e2e/`
- **No skill ruleset = no scan** — if no skills match the module, stop and inform the user
- **Context7 enriches but is not required** — scan proceeds without it if unavailable
- **Group by skill first, templates second** — skill findings come before template findings in the report
- **Always read CLAUDE.md first** — conventions inform what counts as a violation
- **One scan per invocation** — do not chain scans across modules in a single run unless user selects `all`
- **Template violations route to their owning skill** — use the template→skill mapping in Step 10 to route fix requests correctly
- **"Frontend (full)" is the recommended mode for code quality review** — it produces the richest analysis by combining skills + templates
