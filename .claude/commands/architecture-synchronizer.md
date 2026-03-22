---
name: architecture-synchronizer
description: >
  Hierarchy consistency enforcer. Scans .claude/templates/, .claude/skills/, and .claude/commands/
  at runtime, detects broken or missing links in the Command → Skill → Template chain, and
  proposes targeted fixes. Solves the drift problem where commands forget new skills and skills
  forget new templates. Always runs in plan mode first — never edits files without user approval.
disable-model-invocation: true
---

# Architecture Synchronizer

You are `/architecture-synchronizer` — a consistency enforcer for the `.claude/` hierarchy. You detect gaps where the Command → Skill → Template chain has broken or missing links, then propose targeted patches. **You never edit files without user approval — always present the gap report first.**

## Hierarchy Model

```
Command (.claude/commands/)
  └── references Skills (.claude/skills/)
        └── references Templates (.claude/templates/)
```

A "gap" is any missing link in this chain:

| Gap type | Description |
|----------|-------------|
| **Template orphan** | A template exists but no skill's `Templates` section lists it |
| **Skill orphan** | A skill exists but no command references or invokes it |
| **Stale skill ref** | A command references a skill that no longer exists in `.claude/skills/` |
| **Stale template ref** | A skill references a template that no longer exists in `.claude/templates/` |

---

## Flow

### Step 0: Gather Context (MANDATORY — ALWAYS FIRST)

Read **`CLAUDE.md`** at the project root to understand the repository structure, module prefixes, and the role of each entity type.

Also read the claude-skill-builder command if present at `.claude/commands/claude-skill-builder.md` — it defines the authoritative hierarchy rules (Command → Skill → Template, naming conventions, what each entity type is for).

---

### Step 1: Discover All Entities Dynamically

Use Glob to collect every entity at runtime. **Do not hardcode any names.**

**Templates:**
```
.claude/templates/*.md
```
Extract each template's name from its filename (strip `.md`). Read each template's frontmatter (`name`, `description`) to understand its purpose.

**Skills:**
```
.claude/skills/*/SKILL.md
```
Extract each skill's name from its directory name. Read each `SKILL.md` fully to extract:
- The `## Templates` section — what templates does this skill currently reference?
- The skill's `description` / `## When to Invoke` — what is this skill's domain?

**Commands:**
```
.claude/commands/*.md
```
Extract each command's name from its filename. Read each command file fully to extract:
- All skill names referenced (look for `/skill-name` invocations, skill names in tables, `Invoke` columns, `invoke the relevant skill` instructions)
- The command's `description` — what is this command's purpose?

---

### Step 2: Build the Expected Wiring Map

Using the names and descriptions collected in Step 1, infer the **expected** links:

**Expected Template → Skill wiring:**

For each template, determine which skill(s) should reference it by matching:
1. **Name prefix match** — a template named `frontend-hoc-pattern` should be referenced by skill `frontend-pattern-advisor` (same module prefix, semantically related)
2. **Description match** — read the template's description and the skill's `## When to Invoke` / `description` — if the template describes a pattern the skill enforces, it should be wired
3. **Cross-reference** — if a skill's instructions say "follow the pattern in X template" but X is not in the `## Templates` section, that is a gap

**Expected Skill → Command wiring:**

For each skill, determine which command(s) should reference it by matching:
1. **Module prefix match** — a `frontend-*` skill should be referenced by commands that handle frontend work
2. **Functional overlap** — if a command's description says it helps with "frontend code quality" and a `frontend-*` skill exists that enforces quality rules, the command should reference the skill
3. **Explicit invocation** — if a command's flow says "invoke the relevant skill" without naming specific skills, it may be missing skill references

---

### Step 3: Check Actual Wiring

Compare expected wiring (Step 2) against what was actually found (Step 1):

**For each template:**
- Is it listed in at least one skill's `## Templates` section?
- If not → **Template orphan gap**

**For each skill:**
- Is it referenced (by name or `/skill-name` invocation) in at least one command?
- If not → **Skill orphan gap**

**For each command's skill references:**
- Does each referenced skill actually exist in `.claude/skills/`?
- If a referenced skill is missing → **Stale skill reference gap**

**For each skill's template references:**
- Does each referenced template actually exist in `.claude/templates/`?
- If a referenced template is missing → **Stale template reference gap**

Record every gap with:
- Gap type
- Entity with the gap (file path)
- What is missing or stale
- Suggested fix

---

### Step 4: Enter Plan Mode

**Before presenting any findings, call `EnterPlanMode`.**

All output from this point is a proposal. Never exit plan mode automatically — wait for the user to approve specific fixes.

---

### Step 5: Present the Gap Report

Output a structured report with four sections. Use this format:

---

## Synchronization Gap Report

> **Templates discovered**: N
> **Skills discovered**: N
> **Commands discovered**: N
> **Total gaps found**: N

---

### Section A — Template Orphans
*Templates that exist but are not referenced by any skill*

| Template | Expected owning skill | Suggested fix |
|----------|-----------------------|---------------|
| `frontend-hoc-pattern` | `frontend-pattern-advisor` | Add to skill's `## Templates` section |
| `frontend-zustand-store` | `frontend-state-manager` | Add to skill's `## Templates` section |

**Count**: N orphaned templates

If none: *All templates are referenced by at least one skill.*

---

### Section B — Skill Orphans
*Skills that exist but are not referenced by any command*

| Skill | Expected owning command(s) | Suggested fix |
|-------|---------------------------|---------------|
| `frontend-performance-optimizer` | `architecture-scanner`, `codder` | Add skill reference to command's skill table |
| `frontend-page-architector` | `codder` | Add to codder's Available Skills Reference table |

**Count**: N orphaned skills

If none: *All skills are referenced by at least one command.*

---

### Section C — Stale Skill References
*Commands that reference skills which no longer exist*

| Command | Stale reference | Suggested fix |
|---------|----------------|---------------|
| `codder` | `/frontend-old-service` | Remove stale reference — skill no longer exists |

**Count**: N stale skill references

If none: *All command skill references are valid.*

---

### Section D — Stale Template References
*Skills that reference templates which no longer exist*

| Skill | Stale reference | Suggested fix |
|-------|----------------|---------------|
| `frontend-state-manager` | `frontend-old-context` | Remove stale reference — template no longer exists |

**Count**: N stale template references

If none: *All skill template references are valid.*

---

## Summary

| Section | Gaps found |
|---------|-----------|
| A — Template orphans | N |
| B — Skill orphans | N |
| C — Stale skill refs | N |
| D — Stale template refs | N |
| **Total** | **N** |

**Priority fix**: [The single most impactful gap to resolve first — e.g., "5 template orphans in `frontend-state-manager` mean the skill is missing its full ruleset"]

---

### Step 6: Ask Which Gaps to Fix

After presenting the report, ask the user using `AskUserQuestion`:

> "Which gaps would you like me to fix?"

Options:
- **Fix all gaps automatically** — Apply all suggested fixes across all sections
- **Fix a specific section** — User picks Section A, B, C, or D
- **Fix a specific gap** — User describes which one; apply just that fix
- **Done — report only** — No changes needed; exit

---

### Step 7: Apply Fixes

For each approved fix, edit the relevant file:

**Fixing a Template Orphan (Section A):**

Open the expected owning skill's `SKILL.md`. Find the `## Templates` section. If it doesn't exist, add it at the bottom. Add a row to the table:

```
| `<template-name>` | `.claude/templates/<template-name>.md` |
```

If the skill has no `## Templates` section at all, add:

```markdown
## Templates

| Template | Location |
|----------|----------|
| `<template-name>` | `.claude/templates/<template-name>.md` |
```

**Fixing a Skill Orphan (Section B):**

Open the expected owning command's `.md` file. Find the section where skills are listed (look for a table with columns like `Skill | Invoke | Purpose` or `Skill | Use When`). Add a row for the missing skill.

If the command has no skills table, find the most appropriate location in the flow (typically in "Load Module Skills" or "Available Skills Reference" sections) and add the skill with its name, `/invoke` path, and a one-line purpose derived from the skill's own description.

**Fixing a Stale Skill Reference (Section C):**

Open the command file. Find the line or table row referencing the stale skill name. Remove it entirely.

**Fixing a Stale Template Reference (Section D):**

Open the skill's `SKILL.md`. Find the `## Templates` table row referencing the stale template. Remove that row.

After all fixes are applied, confirm to the user:

> "Fixed N gaps. Re-run `/architecture-synchronizer` to verify the hierarchy is fully in sync."

---

## Rules

- **Always discover entities dynamically** — read `.claude/skills/`, `.claude/templates/`, `.claude/commands/` at runtime; never hardcode names
- **Always enter plan mode before presenting findings** — call `EnterPlanMode` before the gap report
- **Never edit files without user approval** — the report is a proposal; fixes only happen after Step 6
- **Infer expected wiring by name prefix + semantic match** — do not require exact string matching; use judgment when a template clearly belongs to a skill
- **Do not over-wire** — a skill should not reference a template it doesn't actually use; only add links that are genuinely relevant
- **Stale references are higher priority than orphans** — broken links cause errors; missing links cause incompleteness
- **Report all four sections even if empty** — a clean section confirms that area is healthy
- **Do not restructure skills or commands beyond adding/removing references** — this command patches wiring only, not content
- **One fix per entity per run** — if a skill needs 3 templates added, add all 3 in one edit operation

---

## Module Prefix Reference

Use this to infer expected wiring when names don't make it obvious:

| Prefix | Module | Relevant commands |
|--------|--------|-------------------|
| `frontend-` | React frontend (`frontend/`) | `codder`, `architecture-scanner` |
| `backend-` | Python Flask API (`backend/`) | `codder` |
| `e2e-` / `playwright-` | E2E tests (`e2e/`) | `codder` |
| `architecture-` | Cross-cutting / meta | `claude-skill-builder` |
