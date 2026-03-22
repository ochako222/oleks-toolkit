---
name: architecture-agent-builder
description: >
  Build, restructure, and audit Claude Code agents — specialized subagents with
  isolated context, custom tools, and focused system prompts. Use when creating new agents,
  improving existing ones, or auditing agent quality. Agents live in .claude/agents/ as markdown files.
---

# Agent Builder

Build, restructure, and audit Claude Code agents — specialized subagents that run in isolated context windows with custom tool access and focused instructions.

## When This Skill MUST Be Used

**ALWAYS invoke this skill when the user's request involves ANY of these:**

- Creating a new Claude Code agent from scratch
- Restructuring or improving an existing agent
- Auditing an agent for completeness or quality
- Asking "how should I write an agent" or "what makes a good agent"
- Creating a specialized assistant role with restricted tools

**If the user mentions "agent", "subagent", "specialist", or "background worker" in the context of the .claude system — use this skill.**

## Critical Safety Rules

**NEVER create agents with `permissionMode: bypassPermissions` unless the user explicitly requests it and understands the implications.**

Agents with bypassed permissions can execute destructive actions (file deletions, force pushes, database drops) without any user approval. This is dangerous.

**NEVER grant `Write`, `Edit`, or `Bash` tools to read-only agents** — if an agent's purpose is analysis or review, restrict its tools accordingly.

**NEVER create agents that hardcode credentials, API keys, or secrets.**

**What IS safe:**
- Creating new `.md` agent files in `.claude/agents/`
- Granting `Read`, `Grep`, `Glob` to any agent (read-only tools)
- Using `model: sonnet` as default (cost-efficient)
- Setting `maxTurns` to prevent runaway agents

**Safe defaults:**
- `permissionMode: default` (asks user for permission)
- `model: sonnet` (unless user needs higher capability)
- `color: blue` (neutral default)

## Architecture / Overview

Agents are specialized subagents in the Claude Code ecosystem — they differ from skills and commands:

| Aspect | Agent | Skill | Command |
|--------|-------|-------|---------|
| **Purpose** | Specialized assistant with isolated context | Reusable workflow instructions | Flow orchestrator |
| **Location** | `.claude/agents/<name>.md` | `.claude/skills/<name>/SKILL.md` | `.claude/commands/<name>.md` |
| **Context** | Own isolated context window | Main conversation context | Main conversation context |
| **Tools** | Can be restricted to specific tools | Inherits all tools | Inherits all tools |
| **Invocation** | Auto-delegated or explicit by name | Auto-invoked by description match | Manual `/command` or auto |
| **Best for** | Domain-specific roles, isolated tasks | Teaching reusable expertise | Multi-step orchestration |

### When to Create an Agent vs a Skill

- **Agent** — When you need an isolated specialist that runs in its own context with restricted tools (e.g., a code reviewer that can only read, not edit)
- **Skill** — When you need reusable workflow instructions that run in the main conversation (e.g., "how to write a Playwright test")

## Step 1: Ask the User What To Do

**Before doing anything else, ask the user which operation they need:**

| Operation | When to use |
|-----------|-------------|
| **Create** | Build a brand-new agent from scratch |
| **Restructure** | Rewrite an existing agent to match the standard pattern |
| **Audit** | Review an existing agent and report what's missing or weak |

Then proceed to the matching section below.

---

## Operation: CREATE a New Agent

### Phase 1 — Gather Requirements (MANDATORY)

You MUST ask the user ALL of the following questions. Do NOT skip any. Do NOT guess answers. If the user's answer is vague, unclear, or doesn't make sense, ask again with a specific follow-up. Each question builds on the previous — wait for answers before continuing.

**Question 1: Role & Purpose**
> What specialized role does this agent fill? What domain or task does it focus on? (e.g., "code reviewer", "database analyst", "test debugger")

**Question 2: Name**
> What should the agent be called? (MUST be kebab-case — lowercase with hyphens, e.g., `code-reviewer`, `db-analyst`, `test-debugger`)

**Question 3: Tools**
> What tools should this agent have access to? Common options:
> - **Read-only**: `Read, Grep, Glob` (analysis, review)
> - **Read + Execute**: `Read, Grep, Glob, Bash` (can run commands)
> - **Full edit**: `Read, Grep, Glob, Edit, Write, Bash` (can modify files)
> - **Custom**: Specify exact tools, including MCP tools if needed
>
> Also: are there tools this agent should be explicitly DENIED? (use `disallowedTools`)

**Question 4: Model**
> Which model should this agent use?
> - `sonnet` — Fast, cost-efficient, good for most tasks (recommended default)
> - `opus` — Most capable, best for complex reasoning
> - `haiku` — Fastest, cheapest, good for simple/repetitive tasks
> - `inherit` — Use whatever the parent conversation uses

**Question 5: Key Responsibilities**
> List the main things this agent should do when invoked. What is its workflow? What checklist should it follow?

**Question 6: Proactive or On-Demand?**
> Should Claude auto-delegate tasks to this agent when it detects a matching task? Or should it only run when explicitly requested by the user?

**Question 7: Constraints & Safety**
> What should this agent NEVER do? Are there files, directories, or actions that are off-limits?

**Question 8: Example Invocations**
> Give 3-5 example phrases a user might say that should trigger this agent.

### Phase 2 — Build the Agent

After gathering ALL answers, generate the agent file following the **Standard Agent Structure** below. Create the file at:

```
.claude/agents/<agent-name>.md
```

### Phase 3 — Review with User

After writing the file, print a summary of what was created and ask:
> "Does this look correct? Want me to adjust anything?"

Make changes until the user approves.

---

## Operation: RESTRUCTURE an Existing Agent

### Phase 1 — Read the Existing Agent

1. Ask the user which agent to restructure (or accept a path)
2. Read the existing agent file completely
3. Identify all existing content and which sections of the standard pattern it maps to

### Phase 2 — Gap Analysis

Present a table showing:

| Standard Section | Status | Notes |
|-----------------|--------|-------|
| Frontmatter (name) | Present/Missing/Weak | ... |
| Frontmatter (description) | Present/Missing/Weak | ... |
| Frontmatter (tools) | Present/Missing/Weak | ... |
| Frontmatter (model) | Present/Missing/Weak | ... |
| System Prompt / Role | Present/Missing/Weak | ... |
| Workflow Steps | Present/Missing/Weak | ... |
| Responsibilities / Checklist | Present/Missing/Weak | ... |
| Constraints / Safety | Present/Missing/Weak | ... |
| Output Format | Present/Missing/Weak | ... |

### Phase 3 — Ask Questions for Missing Content

For each **Missing** or **Weak** section, ask the user targeted questions to fill the gaps. Do NOT invent content — get it from the user.

### Phase 4 — Rewrite

Rewrite the agent file following the **Standard Agent Structure**, preserving all existing good content and adding the new sections. Show the user a before/after summary.

---

## Operation: AUDIT an Existing Agent

### Phase 1 — Read and Score

1. Read the existing agent file
2. Score each section of the standard pattern (0-2):
   - **0** = Missing entirely
   - **1** = Present but incomplete or poorly structured
   - **2** = Complete and well-structured

### Phase 2 — Report

Print a scorecard:

```
Agent Audit: <agent-name>
═══════════════════════════════════════
Section                    Score  Notes
───────────────────────────────────────
Frontmatter (name)         [0-2]  ...
Frontmatter (description)  [0-2]  ...
Frontmatter (tools)        [0-2]  ...
Frontmatter (model/color)  [0-2]  ...
System Prompt / Role       [0-2]  ...
Workflow Steps             [0-2]  ...
Responsibilities           [0-2]  ...
Constraints / Safety       [0-2]  ...
Output Format              [0-2]  ...
───────────────────────────────────────
Total                      [X/18]
```

### Phase 3 — Recommendations

For each section scoring below 2, provide a specific, actionable recommendation explaining what to add or fix. Ask the user if they want to apply the fixes (which transitions into the RESTRUCTURE operation).

---

## Standard Agent Structure

Every agent file MUST follow this structure. The body after frontmatter is the agent's **system prompt** — it tells the agent who it is and what to do.

```markdown
---
name: <kebab-case-name>
description: <1-2 sentences describing when Claude should delegate to this agent. Include "use proactively" if it should auto-trigger.>
tools: <comma-separated list of tools this agent can use>
model: <sonnet|opus|haiku|inherit>
color: <blue|green|yellow|red>
---

# <Agent Role Title>

You are a <role description>. <1-2 sentences about your specialization and expertise.>

## When Invoked

<Step-by-step workflow the agent follows when triggered:>

1. <First action — gather context, read files, etc.>
2. <Second action — analyze, process, etc.>
3. <Third action — produce output, make changes, etc.>

## Responsibilities

<Checklist of what the agent checks, validates, or produces:>

- <Responsibility 1>
- <Responsibility 2>
- <Responsibility 3>

## Constraints

<What the agent must NEVER do:>

- **Never** <constraint 1>
- **Never** <constraint 2>

## Output Format

<How the agent should organize and present its results:>

- <Format guideline 1>
- <Format guideline 2>
```

### Optional Frontmatter Fields

These fields are available but not always needed:

| Field | Type | When to Use |
|-------|------|-------------|
| `disallowedTools` | list | Explicitly deny specific tools |
| `permissionMode` | string | `default`, `acceptEdits`, `dontAsk`, `plan` |
| `maxTurns` | integer | Limit agent's agentic turns to prevent runaway execution |
| `skills` | list | Preload specific skills into agent's context |
| `mcpServers` | object | Grant access to specific MCP servers |
| `hooks` | object | Lifecycle hooks (`PreToolUse`, `PostToolUse`, `Stop`) |
| `memory` | string | Persistent memory scope: `user`, `project`, or `local` |

---

## Configuration / Key Locations

```
.claude/
├── agents/                          # Agent files (flat .md files)
│   ├── code-reviewer.md             # Code quality review agent
│   ├── mobile-responsive.md         # Frontend/responsive design agent
│   ├── playwright-test-generator.md # Test generation with MCP tools
│   ├── playwright-test-healer.md    # Test failure diagnosis
│   └── playwright-test-planner.md   # Test planning
├── commands/                        # Flow descriptions (top layer)
│   ├── claude-skill-builder.md
│   └── codder.md
├── skills/                          # Reusable workflows (middle layer)
│   └── agent-builder/SKILL.md       # THIS SKILL
└── templates/                       # Code examples (bottom layer)
```

**Key behaviors:**
- Agents are **flat `.md` files** (not folders) — simpler than skills
- Agents use YAML frontmatter for metadata and tool configuration
- The body of the file IS the agent's system prompt
- Agent names MUST use kebab-case
- Agents can be scoped: project (`.claude/agents/`) or user (`~/.claude/agents/`)

## Common Tasks

### Creating a Read-Only Analysis Agent

```yaml
---
name: security-scanner
description: Scans code for security vulnerabilities. Use proactively after code changes.
tools: Read, Grep, Glob
model: sonnet
color: red
---
```

### Creating an Agent with MCP Tools

```yaml
---
name: browser-tester
description: Tests web applications using browser automation.
tools: Read, Glob, mcp__playwright-test__browser_navigate, mcp__playwright-test__browser_click, mcp__playwright-test__browser_snapshot
model: sonnet
color: blue
---
```

### Creating an Agent with Hooks

```yaml
---
name: safe-deployer
description: Handles deployment with safety checks.
tools: Read, Bash, Glob
model: opus
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-deploy-command.sh"
---
```

## Troubleshooting

```bash
# List all agents
ls .claude/agents/

# Check agent frontmatter is valid YAML
head -10 .claude/agents/<name>.md

# Verify an agent's tool list
grep "^tools:" .claude/agents/<name>.md

# Check if an agent is referenced by any command or skill
grep -r "agent-name" .claude/commands/ .claude/skills/
```

## Decision Framework

When the user requests an agent:

1. **Is it a specialized role with restricted tools?** → Create an agent in `.claude/agents/`
2. **Is it a reusable workflow with detailed instructions?** → That's a skill, use `architecture-skill-builder` instead
3. **Is it a flow that orchestrates multiple skills?** → That's a command, use `architecture-command-builder` instead
4. **Is it a code example or pattern?** → That's a template, use `architecture-template-builder` instead
5. **Does a similar agent already exist?** → Restructure the existing one instead of creating a duplicate
6. **Should it auto-trigger or be on-demand?** → Include "use proactively" in description for auto-trigger
7. **Unsure?** → Ask the user if they need a specialist role (agent) or a workflow (skill)

## Quality Checklist

Before finalizing ANY agent, verify ALL of the following:

- [ ] **Agent name uses kebab-case** — filename and `name` field are lowercase with hyphens
- [ ] **Frontmatter** has `name`, `description`, `tools`, `model`, and `color`
- [ ] **Description** clearly states when Claude should delegate to this agent
- [ ] **Tools list** matches the agent's role — read-only agents don't get write tools
- [ ] **System prompt** clearly describes the agent's role and expertise
- [ ] **Workflow steps** explain what the agent does when invoked
- [ ] **Responsibilities** list what the agent checks or produces
- [ ] **Constraints** state what the agent must never do
- [ ] **Output format** describes how results should be presented
- [ ] **No placeholder content** — every section has real, useful content
- [ ] **No overly broad tool access** — principle of least privilege

## Important Rules

- **ALWAYS use kebab-case for agent names** — all agent filenames and `name` fields MUST use lowercase with hyphens (e.g., `code-reviewer`, `db-analyst`). NEVER use underscores, camelCase, PascalCase, or spaces. If a user suggests a non-kebab-case name, convert it automatically and inform them.
- **NEVER skip the question phase** — always gather requirements from the user first
- **NEVER invent domain knowledge** — if you don't know something about the user's system, ask
- **If a user answer is vague**, ask a specific follow-up. Example: User says "it reviews code" → ask "What languages? What checklist? Should it auto-fix or only report?"
- **If a user answer doesn't make sense**, say so politely and ask them to clarify
- **Principle of least privilege** — only grant the minimum tools an agent needs for its role
- **Agents go in** `.claude/agents/<agent-name>.md` (flat file, not a folder)
- After writing/modifying an agent, always print the full path so the user knows where it is

## Example Requests

- "Create an agent for code reviews" → Start CREATE operation, ask Phase 1 questions
- "My test-healer agent is messy, clean it up" → Start RESTRUCTURE operation
- "Is my code-reviewer agent any good?" → Start AUDIT operation
- "Build a security scanning agent" → Start CREATE operation, ask Phase 1 questions
- "Create a database analyst subagent" → Start CREATE operation, ask Phase 1 questions
- "Restructure playwright-test-generator to match best practices" → Start RESTRUCTURE operation
