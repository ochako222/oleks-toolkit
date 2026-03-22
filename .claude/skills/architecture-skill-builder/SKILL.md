---
name: architecture-skill-builder
description: >
  Create new Claude Code skills or restructure existing ones to follow
  the standard skill structure pattern. Use when the user wants to build
  a new skill, improve an existing skill, or audit skill quality.
---

# Skill Builder

Build, restructure, and audit Claude Code skills following the proven structure pattern — the gold standard for well-organized, comprehensive skills.

## When This Skill MUST Be Used

**ALWAYS invoke this skill when the user's request involves ANY of these:**

- Creating a new Claude Code skill from scratch
- Restructuring or improving an existing skill's SKILL.md
- Auditing a skill for completeness or quality
- Asking "how should I write a skill" or "what makes a good skill"

## Step 1: Ask the User What To Do

**Before doing anything else, ask the user which operation they need:**

| Operation | When to use |
|-----------|-------------|
| **Create** | Build a brand-new skill from scratch |
| **Restructure** | Rewrite an existing skill to match the standard pattern |
| **Audit** | Review an existing skill and report what's missing or weak |

Then proceed to the matching section below.

---

## Operation: CREATE a New Skill

### Phase 1 — Gather Requirements (MANDATORY)

You MUST ask the user ALL of the following questions. Do NOT skip any. Do NOT guess answers. If the user's answer is vague, unclear, or doesn't make sense, ask again with a specific follow-up. Each question builds on the previous — wait for answers before continuing.

**Question 1: Purpose**
> What is the main purpose of this skill? What problem does it solve or what workflow does it automate?

**Question 2: Name**
> What should the skill be called? (MUST be kebab-case — lowercase with hyphens, e.g., `deploy-staging`, `db-migrate`, `api-scaffold`)

**Question 3: Triggers**
> When should this skill be invoked? List the specific user requests, keywords, or scenarios that should trigger this skill.

**Question 4: Scope & Boundaries**
> What files, directories, or systems does this skill touch? What should it NEVER touch or modify?

**Question 5: Key Operations**
> List the main operations/commands this skill performs. For each one, describe:
> - What it does
> - What inputs it needs
> - What output/result it produces

**Question 6: Safety Concerns**
> Are there any destructive or irreversible actions? What guardrails should be in place?

**Question 7: Example Requests**
> Give 3-5 example phrases a user might say that should trigger this skill.

### Phase 2 — Build the SKILL.md

After gathering ALL answers, generate the SKILL.md following the **Standard Structure Template** below. Create the file at:

```
.claude/skills/<skill-name>/SKILL.md
```

### Phase 3 — Review with User

After writing the file, print a summary of what was created and ask:
> "Does this look correct? Want me to adjust anything?"

Make changes until the user approves.

---

## Operation: RESTRUCTURE an Existing Skill

### Phase 1 — Read the Existing Skill

1. Ask the user which skill to restructure (or accept a path)
2. Read the existing SKILL.md completely
3. Identify all existing content and which sections of the standard pattern it maps to

### Phase 2 — Gap Analysis

Present a table showing:

| Standard Section | Status | Notes |
|-----------------|--------|-------|
| Frontmatter | Present/Missing/Weak | ... |
| Title + Summary | Present/Missing/Weak | ... |
| When MUST Be Used | Present/Missing/Weak | ... |
| Critical Safety Rules | Present/Missing/Weak | ... |
| Architecture/Overview | Present/Missing/Weak | ... |
| Configuration/Locations | Present/Missing/Weak | ... |
| Operations/Patterns | Present/Missing/Weak | ... |
| Common Tasks | Present/Missing/Weak | ... |
| Troubleshooting | Present/Missing/Weak | ... |
| Decision Framework | Present/Missing/Weak | ... |
| Example Requests | Present/Missing/Weak | ... |

### Phase 3 — Ask Questions for Missing Content

For each **Missing** or **Weak** section, ask the user targeted questions to fill the gaps. Do NOT invent content — get it from the user.

### Phase 4 — Rewrite

Rewrite the SKILL.md following the **Standard Structure Template**, preserving all existing good content and adding the new sections. Show the user a before/after summary.

---

## Operation: AUDIT an Existing Skill

### Phase 1 — Read and Score

1. Read the existing SKILL.md
2. Score each section of the standard pattern (0-2):
   - **0** = Missing entirely
   - **1** = Present but incomplete or poorly structured
   - **2** = Complete and well-structured

### Phase 2 — Report

Print a scorecard:

```
Skill Audit: <skill-name>
═══════════════════════════════════════
Section                    Score  Notes
───────────────────────────────────────
Frontmatter                [0-2]  ...
Title + Summary            [0-2]  ...
When MUST Be Used          [0-2]  ...
Critical Safety Rules      [0-2]  ...
Architecture/Overview      [0-2]  ...
Configuration/Locations    [0-2]  ...
Operations/Patterns        [0-2]  ...
Common Tasks               [0-2]  ...
Troubleshooting            [0-2]  ...
Decision Framework         [0-2]  ...
Example Requests           [0-2]  ...
───────────────────────────────────────
Total                      [X/22]
```

### Phase 3 — Recommendations

For each section scoring below 2, provide a specific, actionable recommendation explaining what to add or fix. Ask the user if they want to apply the fixes (which transitions into the RESTRUCTURE operation).

---

## Standard Structure Template

Every skill SKILL.md MUST follow this structure, in this order. Sections can be omitted ONLY if genuinely not applicable (e.g., a read-only skill has no Safety Rules about destructive actions).

```markdown
---
name: <Skill Display Name>
description: >
  <1-3 sentence description loaded into the skill registry.
  Include trigger keywords so Claude knows when to invoke this skill.
  Be specific about what files, systems, or workflows this covers.>
---

# <Skill Name>

<One-line summary of what this skill does and why it exists.>

## When This Skill MUST Be Used

**ALWAYS invoke this skill when the user's request involves ANY of these:**

- <Trigger 1>
- <Trigger 2>
- <Trigger N>

**<Bold closing statement reinforcing when to stop and use this skill.>**

## Critical Safety Rules

**<What to NEVER do, in bold.>**

<Explanation of why, with consequences.>

<Directory tree or file listing showing protected locations, if applicable.>

**<What IS safe to do.>**

**<Safe alternatives to use instead.>**

## Architecture / Overview

<High-level description of the system, project, or domain this skill operates in.>

| Component | Purpose | Location/Config |
|-----------|---------|-----------------|
| ... | ... | ... |

## Operations

<For each major operation the skill performs:>

### <Operation Name>

<Description of what it does.>

**Steps:**
1. <Step with code examples>
2. <Step with code examples>
3. ...

**Important notes:**
- <Gotchas, edge cases, required flags>

## Configuration / Key Locations

<File trees showing relevant directories and files.>

```
<directory-tree>
├── file.ext          # Purpose
├── file2.ext         # Purpose
└── subdir/           # Purpose
```

**Key behaviors:**
- <Auto-reload behavior, restart requirements, etc.>

## Common Tasks

### <Task Category 1>

```bash
<command examples with comments>
```

### <Task Category 2>

<Step-by-step instructions with code blocks.>

## Troubleshooting

```bash
# <Debug command 1 with explanation>
# <Debug command 2 with explanation>
# <Recovery command with explanation>
```

## Decision Framework

When the user requests <domain> changes:

1. **<Condition 1>?** → <Action>
2. **<Condition 2>?** → <Action>
3. **<Condition 3>?** → <Action>
4. **Unsure?** → <Fallback action>

## Example Requests

- "<User says X>" → <What the skill does>
- "<User says Y>" → <What the skill does>
- "<User says Z>" → <What the skill does>
```

---

## Quality Checklist

Before finalizing ANY skill, verify ALL of the following:

- [ ] **Skill name uses kebab-case** — folder name and `name` field are lowercase with hyphens (e.g., `deploy-staging`, NOT `deploy_staging`)
- [ ] **Frontmatter** has `name` and `description` with trigger keywords
- [ ] **Description** is specific enough that Claude can match it to user requests
- [ ] **Triggers** list covers all realistic ways a user might invoke this skill
- [ ] **Safety rules** exist if the skill can modify or delete anything
- [ ] **Code examples** use proper syntax highlighting (```bash, ```typescript, etc.)
- [ ] **File paths** are absolute or clearly relative to a stated base
- [ ] **Tables** are used for structured reference data (not buried in paragraphs)
- [ ] **Directory trees** show file structure where relevant
- [ ] **Bold text** highlights critical warnings and key terms
- [ ] **Decision framework** helps Claude pick the right action without guessing
- [ ] **Example requests** show real phrases users would type
- [ ] **No placeholder content** — every section has real, useful content or is omitted entirely

## Important Rules

- **ALWAYS use kebab-case for skill names** — all skill folder names and `name` fields MUST use lowercase with hyphens (e.g., `deploy-staging`, `db-migrate`, `test-codder`). NEVER use underscores, camelCase, PascalCase, or spaces. If a user suggests a non-kebab-case name, convert it automatically and inform them.
- **NEVER skip the question phase** — always gather requirements from the user first
- **NEVER invent domain knowledge** — if you don't know something about the user's system, ask
- **If a user answer is vague**, ask a specific follow-up. Example: User says "it handles deployments" → ask "Which deployment tool? What environments? What steps does a deploy involve?"
- **If a user answer doesn't make sense**, say so politely and ask them to clarify. Do not proceed with confusing information
- **Preserve existing good content** when restructuring — don't discard well-written sections just to reformat them
- **Skills go in** `.claude/skills/<skill-name>/SKILL.md` within the project
- After writing/modifying a skill, always print the full path so the user knows where it is

## Example Requests

- "Create a new skill for database migrations" → Start CREATE operation, ask Phase 1 questions
- "My deploy skill is messy, clean it up" → Start RESTRUCTURE operation on the deploy skill
- "Is my commit-message skill any good?" → Start AUDIT operation on commit-message skill
- "Build a skill for running tests" → Start CREATE operation, ask Phase 1 questions
- "Restructure manage-folders to match the omarchy pattern" → Start RESTRUCTURE on manage-folders
