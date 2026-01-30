# ORC Plugin Refactoring Plan: 6/10 → 10/10

## Objective
Refactor ORC to be a fully Claude Code marketplace-compliant plugin while preserving all existing functionality and file references.

## Current Issues

| Category | Current | Issue |
|----------|---------|-------|
| **Manifest** | 4/10 | `plugin.json` in root (should be `.claude-plugin/plugin.json`) |
| **Skills** | 5/10 | Uses custom `<orc>` tag, not standard frontmatter |
| **Hooks** | 3/10 | Custom events (story_complete) vs Claude Code events |
| **Commands** | 7/10 | Missing frontmatter fields (description, argument-hint, allowed-tools) |
| **Agents** | 8/10 | Good format, but nested structure (core/specialists) |
| **Documentation** | 9/10 | Excellent |

## Critical Constraints (DO NOT BREAK)

- 13 command names/signatures (`/orc analyze`, `/orc plan`, etc.)
- `.orc/` runtime directory structure
- `state.json` schema (resume capability)
- Agent spawning relationships (names, not paths)
- JSON contract schema references
- Pattern file references in agents

---

## Target Structure (Matches APL Plugin Pattern)

```
ORC/
├── .claude-plugin/
│   └── plugin.json           # NEW: Standard manifest (only file here)
├── agents/                   # FLATTEN: Remove core/specialists nesting
│   ├── orchestrator.md
│   ├── analyzer.md
│   ├── planner.md
│   ├── implementer.md
│   ├── validator.md
│   ├── reviewer.md
│   ├── learner.md
│   ├── architect.md
│   ├── frontend.md
│   ├── backend.md
│   ├── ... (19 total)
├── commands/                 # UPDATE: Add frontmatter to each file
│   ├── analyze.md
│   ├── plan.md
│   ├── ... (13 total)
├── contracts/                # UNCHANGED
├── patterns/                 # UNCHANGED
├── docs/                     # UNCHANGED
├── hooks/
│   ├── hooks.json            # UPDATE: Add Claude Code events
│   └── scripts/              # UNCHANGED
├── skills/                   # NEW: Move SKILL.md here
│   └── orc/
│       └── SKILL.md          # TRANSFORM: Remove <orc> tags, add frontmatter
├── README.md                 # UPDATE: Document new structure
├── CLAUDE.md                 # UPDATE: Document new structure
└── .orc/                     # Runtime (gitignored) - UNCHANGED
```

**Key insight:** The entire repo gets installed as the plugin. `.claude-plugin/` only contains the manifest.

---

## Implementation Phases

### Phase 1: Create Manifest Directory

```bash
mkdir -p .claude-plugin
```

### Phase 2: Create Standard Manifest

**File:** `.claude-plugin/plugin.json`

```json
{
  "name": "orc",
  "displayName": "ORC - Multi-Agent Orchestration System",
  "version": "1.0.0",
  "description": "Autonomous multi-agent system that transforms high-level goals into complete, tested implementations through structured planning and quality gates.",
  "author": {
    "name": "Dakota Smith"
  },
  "repository": "https://github.com/dakotasmith/ORC",
  "homepage": "https://github.com/dakotasmith/ORC#readme",
  "license": "MIT",
  "keywords": [
    "orchestration",
    "multi-agent",
    "autonomous",
    "code-generation",
    "planning",
    "quality-gates"
  ],
  "skills": ["skills/orc/SKILL.md"],
  "commands": [
    "commands/analyze.md",
    "commands/plan.md",
    "commands/show.md",
    "commands/approve.md",
    "commands/run.md",
    "commands/next.md",
    "commands/stop.md",
    "commands/resume.md",
    "commands/retry.md",
    "commands/patterns.md",
    "commands/learn.md",
    "commands/status.md",
    "commands/clear.md"
  ],
  "hooks": "hooks/hooks.json"
}
```

### Phase 3: Transform SKILL.md

**From:** `/SKILL.md` (with `<orc>` tags)
**To:** `/skills/orc/SKILL.md` (with frontmatter)

```bash
mkdir -p skills/orc
```

**New format:**
```yaml
---
name: orc
description: "Multi-Agent Orchestration System for Autonomous Software Development. Invoke with /orc commands to plan, execute, and review code implementations."
version: 1.0.0
triggers:
  - "/orc"
---

# ORC - Multi-Agent Orchestration System

## Overview
[Content from SKILL.md with <orc> tags removed]
...
```

**Changes:**
1. Add YAML frontmatter with name, description, version, triggers
2. Remove opening `<orc>` tag
3. Remove closing `</orc>` tag
4. Keep all other content intact

### Phase 4: Flatten Agents Directory

Move agents from nested structure to flat:

```bash
# Move core agents
mv agents/core/orchestrator.md agents/orchestrator.md
mv agents/core/analyzer.md agents/analyzer.md
mv agents/core/planner.md agents/planner.md
mv agents/core/implementer.md agents/implementer.md
mv agents/core/validator.md agents/validator.md
mv agents/core/reviewer.md agents/reviewer.md
mv agents/core/learner.md agents/learner.md

# Move specialist agents
mv agents/specialists/architect.md agents/architect.md
mv agents/specialists/frontend.md agents/frontend.md
mv agents/specialists/backend.md agents/backend.md
mv agents/specialists/fullstack.md agents/fullstack.md
mv agents/specialists/database.md agents/database.md
mv agents/specialists/security.md agents/security.md
mv agents/specialists/devops.md agents/devops.md
mv agents/specialists/qa.md agents/qa.md
mv agents/specialists/product-designer.md agents/product-designer.md
mv agents/specialists/biz-analyst.md agents/biz-analyst.md
mv agents/specialists/content-strategist.md agents/content-strategist.md
mv agents/specialists/design-researcher.md agents/design-researcher.md

# Remove empty directories
rmdir agents/core agents/specialists
```

**No changes to agent file content needed** - they use agent names (not paths) for spawning:
```yaml
can_spawn: [analyzer, planner, implementer]  # Names, not paths ✓
spawned_by: [implementer, planner]           # Names, not paths ✓
```

**Contract references remain valid** - agents reference contracts by path from repo root:
```markdown
{ "$ref": "contracts/story.schema.json" }  # Still valid ✓
```

### Phase 5: Add Command Frontmatter

Add YAML frontmatter to each command file. **Prepend to existing content, don't replace.**

**Template:**
```yaml
---
description: "Brief description for /help"
argument-hint: "<required> [optional]"
allowed-tools: [Tool1, Tool2, ...]
---

[Existing content unchanged]
```

**All 13 commands:**

| File | description | argument-hint | allowed-tools |
|------|-------------|---------------|---------------|
| `commands/analyze.md` | Analyze codebase conventions and patterns | `[--force] [--focus <areas>]` | `[Read, Glob, Grep, Bash]` |
| `commands/plan.md` | Create or append epic to execution plan | `<goal>` | `[Read, Write, Glob, Grep, Bash, Task]` |
| `commands/show.md` | Display plan summary, epic details, or deviations | `[<epic-id>] [deviations]` | `[Read]` |
| `commands/approve.md` | Approve pending epics for execution | `[<epic-id>]` | `[Read, Write]` |
| `commands/run.md` | Execute all approved epics | `[<epic-id>]` | `[Read, Write, Edit, Glob, Grep, Bash, Task]` |
| `commands/next.md` | Execute next priority epic only | `` | `[Read, Write, Edit, Glob, Grep, Bash, Task]` |
| `commands/stop.md` | Stop execution gracefully at story boundary | `` | `[Read, Write]` |
| `commands/resume.md` | Resume execution from last checkpoint | `` | `[Read, Write, Edit, Glob, Grep, Bash, Task]` |
| `commands/retry.md` | Retry a blocked story | `<story-id>` | `[Read, Write, Edit, Glob, Grep, Bash, Task]` |
| `commands/patterns.md` | Show learned patterns and anti-patterns | `` | `[Read]` |
| `commands/learn.md` | Force pattern extraction now | `` | `[Read, Write, Glob, Grep]` |
| `commands/status.md` | Show current execution state summary | `` | `[Read]` |
| `commands/clear.md` | Clear plan and state (preserves learnings) | `` | `[Read, Write, Bash]` |

### Phase 6: Update Hooks Configuration

**File:** `hooks/hooks.json`

Add Claude Code standard events while preserving ORC internal events:

```json
{
  "hooks": [
    {
      "event": "Stop",
      "command": "hooks/scripts/graceful-shutdown.sh",
      "timeout": 30000
    }
  ],
  "orcEvents": {
    "_comment": "ORC-specific events triggered internally by Orchestrator agent",
    "story_complete": {
      "script": "hooks/scripts/validate-story.sh",
      "timeout_ms": 60000,
      "on_failure": "warn"
    },
    "feature_complete": {
      "script": "hooks/scripts/run-tests.sh",
      "timeout_ms": 300000,
      "on_failure": "block"
    },
    "epic_complete": {
      "script": "hooks/scripts/extract-learnings.sh",
      "timeout_ms": 120000,
      "on_failure": "warn"
    }
  },
  "environment": {
    "ORC_SESSION_ID": "Current session UUID",
    "ORC_PHASE": "Current phase (plan, execute, review, learn)",
    "ORC_EPIC_ID": "Current epic ID",
    "ORC_FEATURE_ID": "Current feature ID",
    "ORC_STORY_ID": "Current story ID",
    "ORC_STATE_DIR": "Path to .orc directory"
  }
}
```

**Create graceful-shutdown.sh:**
```bash
#!/bin/bash
# hooks/scripts/graceful-shutdown.sh
# Called when Claude Code session ends - saves checkpoint

if [ -d ".orc" ]; then
  echo "ORC: Saving checkpoint on shutdown..."
  # Checkpoint logic handled by orchestrator agent
fi
```

### Phase 7: Cleanup Old Files

After all transformations complete:

```bash
# Remove old manifest
rm plugin.json

# Remove old SKILL.md
rm SKILL.md
```

### Phase 8: Update Documentation

**README.md** - Update project structure section
**CLAUDE.md** - Update directory references

---

## File Reference Validation

### References That Stay Valid (No Changes Needed)

| Reference Type | Example | Why It Works |
|----------------|---------|--------------|
| Agent spawning | `can_spawn: [analyzer, planner]` | Uses names, not paths |
| Contract refs | `"$ref": "contracts/story.schema.json"` | Path from repo root unchanged |
| Pattern refs | `Read patterns/auth/jwt-tokens.md` | Path from repo root unchanged |
| Hook scripts | `hooks/scripts/validate-story.sh` | Path from repo root unchanged |
| Runtime dir | `.orc/plan/state.json` | Created at repo root unchanged |

### References That Need Updates

| File | Old Reference | New Reference |
|------|---------------|---------------|
| `SKILL.md` location | `/SKILL.md` | `/skills/orc/SKILL.md` |
| Agent locations | `agents/core/*.md` | `agents/*.md` |
| Agent locations | `agents/specialists/*.md` | `agents/*.md` |

**Note:** Agent content doesn't change - only file locations. Spawning uses names.

---

## Verification Checklist

### Structural Validation
- [ ] `.claude-plugin/plugin.json` exists and is valid JSON
- [ ] `skills/orc/SKILL.md` exists with valid frontmatter
- [ ] All 19 agents in `agents/` (flat, no subdirectories)
- [ ] All 13 commands in `commands/` have frontmatter
- [ ] `hooks/hooks.json` has both Claude Code and ORC events
- [ ] Old files removed (`plugin.json`, `SKILL.md` at root)
- [ ] Old directories removed (`agents/core/`, `agents/specialists/`)

### Functional Validation
- [ ] `/orc analyze` - Creates `.orc/plan/codebase_profile.json`
- [ ] `/orc plan "test"` - Creates plan with epics/features/stories
- [ ] `/orc show` - Displays plan correctly
- [ ] `/orc approve` - Updates state to approved
- [ ] `/orc run` - Executes stories, triggers hooks
- [ ] `/orc stop` - Saves checkpoint
- [ ] `/orc resume` - Restores from checkpoint
- [ ] `/orc status` - Shows current state
- [ ] `/orc clear` - Resets state, preserves learnings

### Reference Validation
- [ ] Agents can read `contracts/*.json` schemas
- [ ] Agents can read `patterns/**/*.md` files
- [ ] Hooks scripts execute from `hooks/scripts/`
- [ ] Runtime state saves to `.orc/`

---

## Files Summary

### Create New
| File | Description |
|------|-------------|
| `.claude-plugin/plugin.json` | Standard manifest |
| `skills/orc/SKILL.md` | Transformed skill with frontmatter |
| `hooks/scripts/graceful-shutdown.sh` | Claude Code Stop hook |

### Move (Flatten)
| From | To |
|------|-----|
| `agents/core/*.md` (7 files) | `agents/*.md` |
| `agents/specialists/*.md` (12 files) | `agents/*.md` |

### Modify In-Place
| File | Change |
|------|--------|
| `commands/analyze.md` | Add frontmatter |
| `commands/plan.md` | Add frontmatter |
| `commands/show.md` | Add frontmatter |
| `commands/approve.md` | Add frontmatter |
| `commands/run.md` | Add frontmatter |
| `commands/next.md` | Add frontmatter |
| `commands/stop.md` | Add frontmatter |
| `commands/resume.md` | Add frontmatter |
| `commands/retry.md` | Add frontmatter |
| `commands/patterns.md` | Add frontmatter |
| `commands/learn.md` | Add frontmatter |
| `commands/status.md` | Add frontmatter |
| `commands/clear.md` | Add frontmatter |
| `hooks/hooks.json` | Add Claude Code events |
| `README.md` | Update structure docs |
| `CLAUDE.md` | Update structure docs |

### Delete
| File/Directory | Reason |
|----------------|--------|
| `plugin.json` (root) | Replaced by `.claude-plugin/plugin.json` |
| `SKILL.md` (root) | Moved to `skills/orc/SKILL.md` |
| `agents/core/` | Agents moved to `agents/` |
| `agents/specialists/` | Agents moved to `agents/` |

### Unchanged
| Directory | Contents |
|-----------|----------|
| `contracts/` | All 13 JSON schemas |
| `patterns/` | All pattern files |
| `docs/` | All documentation |
| `hooks/scripts/` | Existing hook scripts |

---

## Risk Mitigation

| Risk | Mitigation |
|------|------------|
| Breaking agent spawning | Uses names not paths - no change needed |
| Breaking contract refs | Paths from repo root unchanged |
| Breaking pattern refs | Paths from repo root unchanged |
| Breaking hooks | Scripts stay in same location |
| Breaking commands | Only adding frontmatter, not changing content |
