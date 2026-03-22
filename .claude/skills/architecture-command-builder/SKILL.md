---
name: architecture-command-builder
description: >
  Build, restructure, and audit Claude Code custom commands — flow orchestrators
  that invoke skills in sequence. Use when creating new commands, improving existing ones,
  or auditing command quality. Commands live in .claude/commands/ as markdown files.
---

# Command Builder

Build, restructure, and audit Claude Code custom commands — the top orchestration layer in the Command -> Skill -> Template hierarchy.

## When This Skill MUST Be Used

**ALWAYS invoke this skill when the user's request involves ANY of these:**

- Creating a new custom slash command from scratch
- Restructuring or improving an existing command
- Auditing a command for completeness or quality
- Asking "how should I write a command" or "what makes a good command"
- Creating a flow that orchestrates multiple skills

**If the user mentions "command", "slash command", "workflow", "flow", or "orchestrate skills" — use this skill.**

## Critical Safety Rules

**NEVER create commands with `allowed-tools` that include destructive operations without explicit user approval.**

The `allowed-tools` frontmatter field bypasses permission prompts. Granting tools like `Bash` without restrictions can allow destructive actions to run silently.

**NEVER set `disable-model-invocation: false` on commands that perform destructive operations** — this allows Claude to auto-invoke the command without the user explicitly requesting it.

**NEVER create commands that hardcode credentials, API keys, or secrets.**

**What IS safe:**
- Creating new `.md` command files in `.claude/commands/`
- Setting `disable-model-invocation: true` for commands that should only run manually
- Using `$ARGUMENTS` for dynamic input
- Using `!command` syntax for dynamic context injection

## Architecture / Overview

Commands are the top layer of the Claude Code project hierarchy:

```
Command (orchestrates flow)         <-- THIS LAYER
  └── Skill (reusable workflow)
       └── Template (code examples)
```

| Layer | Purpose | Location | Invocation |
|-------|---------|----------|------------|
| **Command** | Describes a flow, invokes skills in sequence | `.claude/commands/<name>.md` | `/command-name` or auto-invoke |
| **Skill** | Reusable workflow with detailed instructions | `.claude/skills/<name>/SKILL.md` | `/skill-name` or auto-invoke |
| **Template** | Code examples and patterns | `.claude/templates/<name>.md` | Referenced by skills |

### What Commands Are
- **Flow orchestrators** — describe a multi-step workflow that invokes skills
- **User-facing entry points** — users type `/command-name` to trigger them
- **High-level instructions** — tell Claude WHAT to do, skills tell HOW
- **Argument-accepting** — support `$ARGUMENTS` for dynamic input

### What Commands Are NOT
- NOT detailed implementation guides (that's what skills do)
- NOT code examples (that's what templates do)
- NOT configuration files (they're instruction files)

### Official Frontmatter Reference

| Field | Required | Description |
|-------|----------|-------------|
| `name` | No | Display name (lowercase, hyphens, max 64 chars). Defaults to filename. |
| `description` | Recommended | When to use. Claude uses this for auto-invocation. |
| `argument-hint` | No | Shows during autocomplete. E.g., `[issue-number]`, `[filename] [format]` |
| `disable-model-invocation` | No | `true` = manual only (`/name`). Default: `false` |
| `user-invocable` | No | `false` = hidden from `/` menu, Claude-only. Default: `true` |
| `allowed-tools` | No | Comma-separated tools that skip permission prompts |
| `model` | No | Force a model: `sonnet`, `opus`, `haiku` |
| `context` | No | `fork` = run in isolated subagent context |
| `agent` | No | Subagent type when `context: fork` (e.g., `Explore`, `Plan`) |

### Dynamic Context Injection

Commands can inject live data using `!command` syntax:

```markdown
## Current Context
- Branch: !`git branch --show-current`
- Status: !`git status --short`
- Recent commits: !`git log --oneline -5`
```

The shell commands execute BEFORE Claude sees the prompt, replacing placeholders with actual output.

### Argument Handling

```markdown
# All arguments as single string
Fix the issue described in $ARGUMENTS

# Indexed arguments
Migrate $ARGUMENTS[0] from $ARGUMENTS[1] to $ARGUMENTS[2]

# Shorthand
Deploy $0 to $1 environment
```

## Step 1: Ask the User What To Do

**Before doing anything else, ask the user which operation they need:**

| Operation | When to use |
|-----------|-------------|
| **Create** | Build a brand-new command from scratch |
| **Restructure** | Rewrite an existing command to match the standard pattern |
| **Audit** | Review an existing command and report what's missing or weak |

Then proceed to the matching section below.

---

## Operation: CREATE a New Command

### Phase 1 — Gather Requirements (MANDATORY)

You MUST ask the user ALL of the following questions. Do NOT skip any.

**Question 1: Purpose**
> What flow does this command orchestrate? What is the end-to-end workflow?

**Question 2: Name**
> What should the command be called? (lowercase with hyphens for the filename, as per Claude Code convention — e.g., `run-tests`, `deploy-staging`, `create-feature`)

**Question 3: Arguments**
> Does this command accept arguments? If so, what arguments and what do they represent?

**Question 4: Skills Involved**
> Which skills does this command invoke, and in what order? Describe the flow step by step.

**Question 5: Invocation Mode**
> Should this command be manual-only (`/command-name`) or auto-invokable by Claude?

**Question 6: Safety Concerns**
> Does this command perform any destructive or irreversible actions? What guardrails should be in place?

**Question 7: Example Usage**
> Give 3-5 example phrases or `/command args` invocations that show how this command is used.

### Phase 2 — Build the Command

After gathering ALL answers, generate the command following the **Standard Command Structure** below. Create the file at:

```
.claude/commands/<command-name>.md
```

### Phase 3 — Review with User

After writing the file, print a summary and ask:
> "Does this look correct? Want me to adjust anything?"

---

## Operation: RESTRUCTURE an Existing Command

### Phase 1 — Read the Existing Command

1. Ask the user which command to restructure (or accept a path)
2. Read the existing command file completely
3. Identify which sections of the standard pattern it maps to

### Phase 2 — Gap Analysis

Present a table showing:

| Standard Section | Status | Notes |
|-----------------|--------|-------|
| Frontmatter | Present/Missing/Weak | ... |
| Title + Purpose | Present/Missing/Weak | ... |
| Context Section | Present/Missing/Weak | ... |
| Flow Steps | Present/Missing/Weak | ... |
| Rules/Constraints | Present/Missing/Weak | ... |
| Output Format | Present/Missing/Weak | ... |

### Phase 3 — Ask Questions for Missing Content

For each **Missing** or **Weak** section, ask the user targeted questions.

### Phase 4 — Rewrite

Rewrite the command following the **Standard Command Structure**, preserving existing good content.

---

## Operation: AUDIT an Existing Command

### Phase 1 — Read and Score

Score each section (0-2):
- **0** = Missing entirely
- **1** = Present but incomplete
- **2** = Complete and well-structured

### Phase 2 — Report

```
Command Audit: <command-name>
═══════════════════════════════════════
Section                    Score  Notes
───────────────────────────────────────
Frontmatter                [0-2]  ...
Title + Purpose            [0-2]  ...
Context Section            [0-2]  ...
Flow Steps                 [0-2]  ...
Rules/Constraints          [0-2]  ...
Output Format              [0-2]  ...
───────────────────────────────────────
Total                      [X/12]
```

### Phase 3 — Recommendations

For each section scoring below 2, provide actionable recommendations.

---

## Standard Command Structure

Every command MUST follow this structure:

```markdown
---
name: <command-name>
description: >
  <1-2 sentence description of what this command does.
  Include trigger keywords for auto-invocation.>
disable-model-invocation: <true|false>
argument-hint: <[arg1] [arg2]>
---

# <Command Name>

<One-line summary of the end-to-end flow this command orchestrates.>

## Context

<Dynamic context injection using !`command` syntax, if needed.>

- Current branch: !`git branch --show-current`
- Changed files: !`git diff --name-only`

## Flow

**This command orchestrates the following skills in order:**

### Step 1: <Step Name>
**Skill:** `<skill_name>`

<What this step does and what it produces.>

### Step 2: <Step Name>
**Skill:** `<skill_name>`

<What this step does, using output from Step 1.>

### Step 3: <Step Name>
**Skill:** `<skill_name>`

<Final step and what the end result should be.>

## Rules

- <Constraint 1 — what MUST happen>
- <Constraint 2 — what MUST NOT happen>
- <Constraint 3 — edge cases to handle>

## Output

<What the command should produce when finished.>
<Expected format of the final output.>
```

---

## Configuration / Key Locations

```
.claude/
├── commands/                     # Flow descriptions (top layer)
│   ├── run-tests.md              # /run-tests command
│   ├── deploy-staging.md         # /deploy-staging command
│   └── create-feature.md        # /create-feature command
├── skills/                       # Reusable workflows (middle layer)
│   └── <name>/SKILL.md
└── templates/                    # Code examples (bottom layer)
    └── <name>.md
```

**Key behaviors:**
- Commands are **flat `.md` files** (not folders) — one file per command
- Command filenames use **lowercase with hyphens** (Claude Code convention for slash commands)
- Commands create `/command-name` slash commands automatically
- The `description` field controls auto-invocation — be specific with trigger keywords
- `disable-model-invocation: true` prevents Claude from auto-triggering the command

## Common Tasks

### Creating a Simple Manual Command

```markdown
---
name: commit
description: Create a structured git commit
disable-model-invocation: true
---

# Commit

Create a git commit following project conventions.

## Context

- Changed files: !`git diff --name-only`
- Branch: !`git branch --show-current`

## Flow

### Step 1: Analyze Changes
Review the changed files and understand what was modified.

### Step 2: Write Commit Message
Write a clear, descriptive commit message.

### Step 3: Execute Commit
Stage relevant files and create the commit.

## Rules

- Follow conventional commit format
- Never commit .env files or credentials
- Include ticket number if on a feature branch
```

### Creating a Command That Invokes Skills

```markdown
---
name: create-e2e-test
description: Create a complete E2E test with page objects and verification
disable-model-invocation: true
argument-hint: [feature-name]
---

# Create E2E Test

End-to-end workflow for creating a verified Playwright test for $ARGUMENTS.

## Flow

### Step 1: Create Page Objects
**Skill:** `playwright-pom-generator`

Create any missing Page Objects, Components, or Modals needed for the feature.

### Step 2: Write Test
**Skill:** `playwright-test-writer`

Write the test spec following project patterns.

### Step 3: Run and Verify
**Skill:** `test-codder`

Run the test, fix failures, and verify it passes.

## Rules

- Maximum 2 test runs (Write -> Run -> Fix -> Verify pattern)
- All code must follow project standards (double quotes, tabs, no any types)
- Report results to user after completion
```

## Troubleshooting

```bash
# List all commands
ls .claude/commands/

# Check if command is recognized (look for it in skill registry)
# Commands appear as slash commands after creation

# Verify frontmatter is valid YAML
head -10 .claude/commands/<name>.md

# Check which skills a command references
grep -i "skill" .claude/commands/<name>.md
```

## Decision Framework

When the user requests a new command:

1. **Is it a multi-step flow that invokes skills?** -> Create a command in `.claude/commands/`
2. **Is it a single reusable workflow with detailed instructions?** -> That's a skill, use `architecture-skill-builder` instead
3. **Is it a code example or pattern?** -> That's a template, use `architecture-template-builder` instead
4. **Should it accept user input?** -> Use `argument-hint` and `$ARGUMENTS` in the command
5. **Is it dangerous or destructive?** -> Set `disable-model-invocation: true` and add safety rules
6. **Should Claude auto-invoke it?** -> Keep `disable-model-invocation: false` and write a specific `description`
7. **Unsure?** -> Ask the user if they need a flow (command), a workflow (skill), or a code pattern (template)

## Quality Checklist

Before finalizing any command:

- [ ] **Filename uses lowercase-hyphens** (Claude Code convention for slash commands)
- [ ] **Frontmatter** has `name`, `description`, and `disable-model-invocation`
- [ ] **Description** includes trigger keywords for auto-invocation (or `disable-model-invocation: true`)
- [ ] **Flow section** clearly lists steps and which skills they invoke
- [ ] **Rules section** defines constraints and safety guardrails
- [ ] **Output section** describes what the command produces
- [ ] **Context section** uses `!command` injection if dynamic data is needed
- [ ] **Arguments** are documented if the command accepts input
- [ ] **No placeholder content** — every section has real content

## Important Rules

- **Command filenames use lowercase-hyphens** (e.g., `run-tests.md`, `deploy-staging.md`) — this is the Claude Code convention for slash commands that become `/run-tests`, `/deploy-staging`
- **NEVER skip the question phase** — always gather requirements first
- **NEVER invent skill names** — only reference skills that actually exist in `.claude/skills/`
- **Commands describe WHAT to do, skills describe HOW** — keep commands high-level
- **One command = one flow** — don't mix unrelated workflows in one command
- **Commands go in** `.claude/commands/<command-name>.md`
- After writing/modifying a command, always print the full path

## Example Requests

- "Create a command for running E2E tests" -> Start CREATE operation, ask Phase 1 questions
- "My deploy command is messy, clean it up" -> Start RESTRUCTURE operation
- "Is my commit command any good?" -> Start AUDIT operation
- "Build a slash command that creates a feature with tests" -> Start CREATE operation
- "Create a command that orchestrates POM creation + test writing + verification" -> Start CREATE operation

---

**Remember**: Commands are conductors, not musicians. They orchestrate the flow and invoke skills — they don't contain implementation details themselves. Keep commands high-level and let skills do the heavy lifting.
