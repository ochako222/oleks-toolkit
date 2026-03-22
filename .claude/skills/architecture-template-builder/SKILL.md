---
name: architecture-template-builder
description: >
  Build, restructure, and audit Claude Code templates — reusable code example files
  that skills reference for consistent code generation. Use when creating new templates,
  improving existing ones, or auditing template quality. Templates live in .claude/templates/.
---

# Template Builder

Build, restructure, and audit Claude Code templates — the code example layer in the Command -> Skill -> Template hierarchy.

## When This Skill MUST Be Used

**ALWAYS invoke this skill when the user's request involves ANY of these:**

- Creating a new code template from scratch
- Restructuring or improving an existing template
- Auditing a template for completeness or quality
- Asking "how should I write a template" or "what makes a good template"
- Adding code examples that skills should reference

**If the user mentions "template", "code example", "code pattern", "boilerplate", or "scaffold" in the context of the .claude system — use this skill.**

## Critical Safety Rules

**NEVER create templates that contain real credentials, API keys, or secrets.**

Templates are shared code examples. Hardcoded secrets in templates will leak into every file generated from them.

**NEVER modify existing templates without reading them first** — they may be referenced by active skills.

**NEVER create templates that duplicate existing ones** — always check `.claude/templates/` for similar templates before creating new ones.

**What IS safe:**
- Creating new template files in `.claude/templates/`
- Updating code examples in existing templates
- Adding new code blocks to existing templates

## Architecture / Overview

Templates are the bottom layer of the Claude Code project hierarchy:

```
Command (orchestrates flow)
  └── Skill (reusable workflow)
       └── Template (code examples)
```

| Layer | Purpose | Location | Format |
|-------|---------|----------|--------|
| **Command** | Describes a flow, invokes skills | `.claude/commands/<name>.md` | YAML frontmatter + markdown |
| **Skill** | Reusable workflow with instructions | `.claude/skills/<name>/SKILL.md` | YAML frontmatter + markdown |
| **Template** | Code examples and patterns | `.claude/templates/<name>.md` | YAML frontmatter + code blocks |

### What Templates Are
- **Pure code examples** — the actual code patterns Claude should follow when generating code
- **Referenced by skills** — skills point to templates for consistent code output
- **Language-specific** — each template targets a specific language/framework
- **Copy-paste ready** — code blocks should work as-is or with minimal variable substitution

### What Templates Are NOT
- NOT workflow instructions (that's what skills do)
- NOT flow orchestration (that's what commands do)
- NOT documentation or explanations (keep prose minimal, maximize code)

## Step 1: Ask the User What To Do

**Before doing anything else, ask the user which operation they need:**

| Operation | When to use |
|-----------|-------------|
| **Create** | Build a brand-new template from scratch |
| **Restructure** | Rewrite an existing template to match the standard pattern |
| **Audit** | Review an existing template and report what's missing or weak |

Then proceed to the matching section below.

---

## Operation: CREATE a New Template

### Phase 1 — Gather Requirements (MANDATORY)

You MUST ask the user ALL of the following questions. Do NOT skip any. Do NOT guess answers.

**Question 1: Purpose**
> What code pattern does this template provide? What problem does it solve?

**Question 2: Name**
> What should the template be called? (MUST be kebab-case — lowercase with hyphens, e.g., `page-object`, `serial-test-suite`, `api-controller`)

**Question 3: Language/Framework**
> What language and framework is this template for? (e.g., TypeScript + Playwright, React + Ant Design)

**Question 4: Variables**
> What parts of the template should be customizable? List the placeholder variables and what they represent.

**Question 5: Which Skill Uses This?**
> Which skill(s) will reference this template? (e.g., `playwright-pom-generator`, `playwright-test-writer`)

**Question 6: Code Examples**
> Provide the actual code pattern this template should contain, or describe it in enough detail to generate it.

### Phase 2 — Build the Template

After gathering ALL answers, generate the template following the **Standard Template Structure** below. Create the file at:

```
.claude/templates/<template-name>.md
```

### Phase 3 — Review with User

After writing the file, print a summary and ask:
> "Does this look correct? Want me to adjust anything?"

---

## Operation: RESTRUCTURE an Existing Template

### Phase 1 — Read the Existing Template

1. Ask the user which template to restructure (or accept a path)
2. Read the existing template completely
3. Identify which sections of the standard pattern it maps to

### Phase 2 — Gap Analysis

Present a table showing:

| Standard Section | Status | Notes |
|-----------------|--------|-------|
| Frontmatter | Present/Missing/Weak | ... |
| Title + Description | Present/Missing/Weak | ... |
| Variables Reference | Present/Missing/Weak | ... |
| Code Block(s) | Present/Missing/Weak | ... |
| Usage Notes | Present/Missing/Weak | ... |
| Referenced By | Present/Missing/Weak | ... |

### Phase 3 — Ask Questions for Missing Content

For each **Missing** or **Weak** section, ask the user targeted questions to fill the gaps.

### Phase 4 — Rewrite

Rewrite the template following the **Standard Template Structure**, preserving existing good content.

---

## Operation: AUDIT an Existing Template

### Phase 1 — Read and Score

Score each section (0-2):
- **0** = Missing entirely
- **1** = Present but incomplete
- **2** = Complete and well-structured

### Phase 2 — Report

```
Template Audit: <template_name>
═══════════════════════════════════════
Section                    Score  Notes
───────────────────────────────────────
Frontmatter                [0-2]  ...
Title + Description        [0-2]  ...
Variables Reference        [0-2]  ...
Code Block(s)              [0-2]  ...
Usage Notes                [0-2]  ...
Referenced By              [0-2]  ...
───────────────────────────────────────
Total                      [X/12]
```

### Phase 3 — Recommendations

For each section scoring below 2, provide actionable recommendations.

---

## Standard Template Structure

Every template MUST follow this structure:

```markdown
---
name: <template_name>
description: >
  <1-2 sentence description of what code pattern this template provides.
  Include the language, framework, and what type of code it generates.>
language: <typescript|javascript|bash|python|etc>
referenced_by:
  - <skill_name_1>
  - <skill_name_2>
---

# <Template Name>

<One-line description of the code pattern this template provides.>

## Variables

| Variable | Type | Description | Example |
|----------|------|-------------|---------|
| `<VARIABLE_NAME>` | string | What this variable represents | `LoginPage` |
| `<ANOTHER_VAR>` | string | What this variable represents | `submitButton` |

## Template

\```typescript
// The actual code pattern with <VARIABLE_NAME> placeholders
// This should be copy-paste ready after variable substitution

import { expect } from "@playwright/test";

export class <CLASS_NAME> extends <BASE_CLASS> {
    private <LOCATOR_NAME> = this.page.locator('<SELECTOR>');

    @step("<STEP_DESCRIPTION>")
    async <METHOD_NAME>() {
        await this.<LOCATOR_NAME>.click();
    }
}
\```

## Usage Notes

- <When to use this template>
- <Common modifications>
- <Edge cases or gotchas>

## Referenced By

- [`<skill_name>`](.claude/skills/<skill_name>/SKILL.md) — <how this skill uses the template>
```

---

## Configuration / Key Locations

```
.claude/
├── commands/                     # Flow descriptions (top layer)
│   └── <name>.md
├── skills/                       # Reusable workflows (middle layer)
│   └── <name>/SKILL.md
└── templates/                    # Code examples (bottom layer)
    ├── page-object.md            # Page Object template
    ├── serial-test-suite.md      # Test suite template
    ├── component.md              # Component template
    └── modal.md                  # Modal template
```

**Key behaviors:**
- Templates are flat `.md` files (not folders) — they're simpler than skills
- Templates use YAML frontmatter for metadata and discoverability
- Templates are referenced by skills via relative path or name
- Template names MUST use kebab-case

## Troubleshooting

```bash
# List all templates
ls .claude/templates/

# Check if a template is referenced by any skill
grep -r "template_name" .claude/skills/

# Verify template frontmatter is valid YAML
head -20 .claude/templates/<name>.md
```

## Decision Framework

When the user requests a template:

1. **Is it a reusable code pattern?** -> Create a template in `.claude/templates/`
2. **Is it a workflow with instructions?** -> That's a skill, use `architecture-skill-builder` instead
3. **Is it a flow that orchestrates multiple skills?** -> That's a command, use `architecture-command-builder` instead
4. **Does a similar template already exist?** -> Restructure the existing one instead of creating a duplicate
5. **Unsure?** -> Ask the user if they need a code example (template) or a workflow (skill)

## Quality Checklist

Before finalizing any template:

- [ ] **Template name uses kebab-case**
- [ ] **Frontmatter** has `name`, `description`, `language`, and `referenced_by`
- [ ] **Variables table** lists all placeholders with types and examples
- [ ] **Code block** uses correct syntax highlighting for the language
- [ ] **Code is copy-paste ready** after variable substitution
- [ ] **Usage notes** explain when and how to use the template
- [ ] **Referenced by** links to the skills that use this template
- [ ] **No placeholder content** — every code block has real, working patterns

## Important Rules

- **ALWAYS use kebab-case for template names** — e.g., `page-object`, `serial-test-suite`, `api-controller`
- **NEVER skip the question phase** — always gather requirements first
- **NEVER invent code patterns** — if you don't know the project's conventions, read existing code first
- **Templates are CODE, not instructions** — keep prose minimal, maximize code blocks
- **One template = one pattern** — don't mix multiple unrelated patterns in one template
- **Templates go in** `.claude/templates/<template-name>.md`
- After writing/modifying a template, always print the full path

## Example Requests

- "Create a template for page objects" -> Start CREATE operation, ask Phase 1 questions
- "My test suite template is outdated, fix it" -> Start RESTRUCTURE operation
- "Is my component template complete?" -> Start AUDIT operation
- "Add a template for modal classes" -> Start CREATE operation, ask Phase 1 questions
- "Create a boilerplate for API controllers" -> Start CREATE operation, ask Phase 1 questions
