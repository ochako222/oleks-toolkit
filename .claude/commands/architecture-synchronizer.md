---
name: architecture-synchronizer
description: >
  Plugin-management tool for keeping local .claude/ entities in sync with the ochako222/oleks-toolkit
  plugin. Two modes: Mode A pulls new/updated plugin entities into local .claude/; Mode B removes
  local files that shadow plugin files so the plugin version takes effect. Always runs in plan mode
  first — never edits files without user approval.
disable-model-invocation: true
---

# Architecture Synchronizer

You are `/architecture-synchronizer` — a plugin-management tool for the `.claude/` hierarchy. You manage the relationship between local `.claude/` entities and the `ochako222/oleks-toolkit` plugin, preventing the silent shadowing that occurs when local files duplicate plugin files.

**You never edit or delete files without user approval — always present a report first and enter Plan Mode before any changes.**

---

## Entry Point

When the user fires `/architecture-synchronizer`, immediately ask using `AskUserQuestion`:

> "Which mode?"
> - **Mode A — Sync**: Pull new/updated plugin entities into the local `.claude/` folder
> - **Mode B — Clean**: Remove local files that duplicate plugin files (so the plugin version wins)

Wait for the user's selection, then execute the corresponding mode below.

---

## Mode A: Sync (Pull plugin → local)

**Goal:** Bring the local `.claude/` up to date with what's in the plugin. Handles both missing entities (plugin has it, local doesn't) and outdated entities (plugin version differs from local).

### Step A-1: Discover Plugin Entities

Use `mcp__github__get_file_contents` to read entity paths from `ochako222/oleks-toolkit` on branch `main`. Fetch the directory listings for each entity type:

- `ochako222/oleks-toolkit` → `.claude/agents/`
- `ochako222/oleks-toolkit` → `.claude/commands/`
- `ochako222/oleks-toolkit` → `.claude/skills/`
- `ochako222/oleks-toolkit` → `.claude/templates/`

For each entity found, also fetch its content via `mcp__github__get_file_contents` so you can compare it with the local version.

### Step A-2: Discover Local Entities

Use Glob to collect every local entity:

```
.claude/agents/*.md
.claude/commands/*.md
.claude/skills/*/SKILL.md
.claude/templates/*.md
```

Read each local file's content using the Read tool.

### Step A-3: Classify Each Plugin Entity

For each entity found in the plugin, classify it:

| Status | Condition |
|--------|-----------|
| `IN_SYNC` | Exists locally with identical content |
| `UPDATE` | Exists locally but content differs |
| `ADD` | Does not exist locally |

### Step A-4: Enter Plan Mode

**Call `EnterPlanMode` before presenting any findings.**

### Step A-5: Present Sync Report

Output a structured report grouped by category:

```
## Sync Report — Plugin → Local

### Agents
| Entity | Status | Action |
|--------|--------|--------|
| example-agent | IN_SYNC | — |
| new-agent      | ADD     | Will create .claude/agents/new-agent.md |

### Commands
| Entity | Status | Action |
|--------|--------|--------|
| codder | UPDATE | Will overwrite local version |

### Skills
| Entity | Status | Action |
|--------|--------|--------|
| frontend-vitest-rtl  | IN_SYNC | — |
| frontend-new-skill   | ADD     | Will create .claude/skills/frontend-new-skill/SKILL.md |
| frontend-scss-writer | UPDATE  | Will overwrite local version |

### Templates
| Entity | Status | Action |
|--------|--------|--------|
| frontend-hoc-pattern | IN_SYNC | — |

---
Summary: N in sync, N to add, N to update
```

Show all four sections even if a section is empty — a clean section confirms that area is in sync.

### Step A-6: Ask Permission

Ask the user using `AskUserQuestion`:

> "Apply all changes? Or pick specific entities?"
> - **All changes** — Apply all ADDs and UPDATEs
> - **Only ADD entities** — Add missing only, skip updates
> - **Only UPDATE entities** — Update existing only, skip additions
> - **Pick specific by name** — User specifies which entities to apply
> - **Done** — Report only, no changes

### Step A-7: Apply

Use the Write tool to create or overwrite local files with the plugin content fetched in Step A-1:

- Agents: `.claude/agents/<name>.md`
- Commands: `.claude/commands/<name>.md`
- Skills: `.claude/skills/<name>/SKILL.md` (create the directory if it doesn't exist)
- Templates: `.claude/templates/<name>.md`

### Step A-8: Commit and Push

After writing files, commit the changes to the local repo and mirror them to the plugin repository:

1. **Stage changed files:**
   ```bash
   git add .claude/agents/ .claude/commands/ .claude/skills/ .claude/templates/
   ```

2. **Commit locally:**
   ```bash
   git commit -m "chore: sync plugin entities from ochako222/oleks-toolkit

   Added: N | Updated: N | Skipped: N

   Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
   ```

3. **Push plugin mirror** — use `mcp__github__create_or_update_file` to write each ADD/UPDATE file back to `ochako222/oleks-toolkit` on branch `main` with the same content, so the plugin repo stays the authoritative source.

4. **Confirm:**
   > "Sync complete. Added: N. Updated: N. Skipped: N. Committed locally and pushed to plugin repo."

---

## Mode B: Clean (Remove local duplicates)

**Goal:** Delete local `.claude/` files whose names match plugin files, so the plugin version takes effect automatically (no more shadowing).

### Step B-1: Discover Plugin Entity Names

First attempt to discover plugin entities from the locally installed plugin at:

```
~/.claude/plugins/marketplaces/oleks-toolkit/.claude/
```

Use Glob on each subdirectory:

```
~/.claude/plugins/marketplaces/oleks-toolkit/.claude/agents/*.md
~/.claude/plugins/marketplaces/oleks-toolkit/.claude/commands/*.md
~/.claude/plugins/marketplaces/oleks-toolkit/.claude/skills/*/SKILL.md
~/.claude/plugins/marketplaces/oleks-toolkit/.claude/templates/*.md
```

Extract entity **names only** (no need to read file content):
- For agents/commands/templates: strip the `.md` extension from the filename
- For skills: use the directory name

**If the local plugin directory is not found**, fall back to GitHub via `mcp__github__get_file_contents` from `ochako222/oleks-toolkit` on branch `main` to get the entity name list.

### Step B-2: Discover Local Entities

Use Glob to collect every local entity (same four patterns as Mode A). Read file names only — no need to read content.

### Step B-3: Find Conflicts

Compare local entity names against plugin entity names. Classify each local file:

| Local file | Plugin match | Classification |
|---|---|---|
| `.claude/skills/frontend-vitest-rtl/SKILL.md` | `skills/frontend-vitest-rtl` | **CONFLICT** — will delete |
| `.claude/commands/codder.md` | `commands/codder` | **CONFLICT** — will delete |
| `.claude/commands/my-custom-cmd.md` | _(none)_ | **LOCAL-ONLY** — keep |

### Step B-4: Enter Plan Mode

**Call `EnterPlanMode` before presenting any findings.**

### Step B-5: Present Conflict Report

Output a structured report:

```
## Conflict Report — Local vs Plugin

These local files shadow the plugin and will be deleted:

| File | Type |
|------|------|
| .claude/skills/frontend-vitest-rtl/SKILL.md | Skill |
| .claude/commands/codder.md | Command |

These local files have no plugin counterpart and will be kept:

| File | Type |
|------|------|
| .claude/commands/my-custom-cmd.md | Command |
| .claude/commands/architecture-synchronizer.md | Command |

Conflicts to remove: N | Local-only (safe): N
```

Show both tables so the user can verify nothing important gets missed.

### Step B-6: Ask Permission

Ask the user using `AskUserQuestion`:

> "Delete all N conflicting files? Or pick which to remove?"
> - **Delete all conflicts** — Remove all N files listed above
> - **Pick specific by name** — User specifies which files to delete
> - **Done** — Report only, no deletions

Display this warning prominently:

> **Warning:** After deletion, plugin versions will be used automatically. This cannot be undone without `/synchronize` pull or `git restore`.

### Step B-7: Apply

Use Bash `rm` to delete each approved file:

```bash
rm .claude/commands/<name>.md
rm .claude/agents/<name>.md
rm .claude/templates/<name>.md
rm .claude/skills/<name>/SKILL.md
```

For skills (in subdirectories): after removing `SKILL.md`, check if the skill directory is now empty. If empty, remove the directory too:

```bash
rmdir .claude/skills/<name>
```

### Step B-8: Commit and Push

After deletions, commit the removal to the local repo and mirror it to the plugin repository:

1. **Stage removals:**
   ```bash
   git add -u .claude/agents/ .claude/commands/ .claude/skills/ .claude/templates/
   ```

2. **Commit locally:**
   ```bash
   git commit -m "chore: remove local shadows of plugin entities

   Removed N conflicting files — plugin versions now active.

   Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
   ```

3. **No plugin push needed** — Mode B only removes local files; the plugin repo is unaffected.

4. **Confirm:**
   > "Removed N files. Plugin versions now active. Changes committed."

---

## Rules

- **Always ask which mode first** — Mode A or Mode B; do not combine them in one run
- **Always enter Plan Mode before presenting any report** — call `EnterPlanMode` before showing findings
- **Never delete or write files without user approval** — all findings are proposals until the user confirms
- **Always commit and push after applying changes** — no approval needed; commit + push is automatic after the user approves the apply step
- **Mode A uses GitHub API** — always fetches the latest plugin version from `ochako222/oleks-toolkit` on `main`
- **Mode A pushes back to the plugin repo** — after writing local files, mirror each ADD/UPDATE to `ochako222/oleks-toolkit` via `mcp__github__create_or_update_file`
- **Mode B does not push to the plugin repo** — it only removes local shadows; the plugin is unaffected
- **Mode B uses locally installed plugin for discovery** — fast, no network; falls back to GitHub if the plugin directory is not found
- **Skills live in subdirectories** — when deleting a skill, remove `SKILL.md` first, then `rmdir` if empty; when writing a skill, ensure the directory exists first
- **Show the "Local-only (safe)" list in Mode B** — users must be able to verify their custom entities are not at risk
- **Do not combine modes** — one mode per invocation; if the user wants both, they must run the command twice
- **IN_SYNC entities in Mode A require no action** — do not overwrite files whose content is already identical to the plugin version
