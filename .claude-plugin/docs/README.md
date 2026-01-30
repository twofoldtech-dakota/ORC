# ORC (Orchestrator)

> Multi-Agent Orchestration System for Autonomous Software Development

ORC is a Claude Code plugin that transforms high-level goals into complete, tested implementations through a structured phased approach with mandatory plan approval.

## Quick Start

```bash
# Create a plan
/orc plan "Build REST API with user authentication"

# Review the plan
/orc show

# Approve the plan
/orc approve

# Execute
/orc run
```

## Core Philosophy

**Quality > Capability > Developer Experience > Speed > Cost**

ORC follows a strict workflow:

1. **Plan Phase** - Decompose goal into Epic → Feature → Story hierarchy
2. **User Approval** - Mandatory review before any implementation
3. **Execute Phase** - Implement with ReAct pattern, validate each story
4. **Review Phase** - Final quality gates and deviation analysis
5. **Learn Phase** - Extract patterns for future use

## Key Features

### Plan-First Execution
No code is written until you approve the plan. Review exactly what will be built before it happens.

### Self-Validation
Every story has explicit acceptance criteria verified with evidence. No GUI required—the system validates itself.

### Strict Approach Adherence
The Implementer follows suggested approaches exactly. Any deviation is documented, categorized, and flagged for review.

### Autonomous Recovery
Failed stories retry with different approaches (up to 3 attempts). If still failing, they're marked blocked and non-dependent work continues.

### Parallel Execution
Independent stories execute concurrently. Dependencies are analyzed at plan time to prevent conflicts.

### Pattern Learning
Successful implementations are extracted as patterns. Future similar tasks automatically receive proven approaches.

## Commands

### Planning
| Command | Description |
|---------|-------------|
| `/orc plan <goal>` | Create or append to plan |
| `/orc show` | Display plan summary |
| `/orc show <epic-id>` | Display epic details |
| `/orc show deviations` | Show deviations for review |
| `/orc approve` | Approve all pending epics |
| `/orc approve <epic-id>` | Approve specific epic |

### Execution
| Command | Description |
|---------|-------------|
| `/orc run` | Execute all approved epics |
| `/orc run <epic-id>` | Execute specific epic |
| `/orc next` | Execute next priority epic only |
| `/orc stop` | Stop execution gracefully |
| `/orc resume` | Resume from last checkpoint |
| `/orc retry <story-id>` | Retry a blocked story |

### Learning & Utility
| Command | Description |
|---------|-------------|
| `/orc patterns` | Show learned patterns |
| `/orc learn` | Force pattern extraction |
| `/orc status` | Show current state |
| `/orc clear` | Clear plan and state |

## Plan Hierarchy

```
Plan
└── Epic (self-contained project milestone)
    └── Feature (self-contained capability)
        └── Story (atomic task with acceptance criteria)
```

**Rules:**
- Epics are self-contained (no cross-epic dependencies)
- Features are self-contained (no cross-feature dependencies)
- Stories can depend on stories within the same feature only

## Quality Gates

### Story Completion
- ✓ All acceptance criteria verified
- ✓ All unit tests pass
- ✓ Type checking passes
- ✓ No lint errors
- ✓ No new security vulnerabilities
- ✓ Approach compliance verified

### Feature Completion
- ✓ All stories completed
- ✓ Integration tests pass
- ✓ Feature-level acceptance criteria verified

### Epic Completion
- ✓ All features completed
- ✓ E2E tests pass
- ✓ Final review completed
- ✓ Full security scan passes

## Agent System

### Core Agents (6)
| Agent | Role |
|-------|------|
| Orchestrator | State management, phase control |
| Planner | Goal decomposition |
| Implementer | Code implementation (ReAct) |
| Validator | Test & acceptance verification |
| Reviewer | Final quality gate (Reflexion) |
| Learner | Pattern extraction |

### Specialists (11)
Spawned on-demand: Architect, Product Designer, DevOps, Security, Database, Frontend, Backend, Full Stack, QA, Biz Analyst, Content Strategist

## Runtime Directory

ORC maintains state in `.orc/`:

```
.orc/
├── plan/
│   ├── state.json           # Execution state
│   ├── plan.json            # Master plan
│   ├── learnings.json       # Patterns
│   ├── embeddings.json      # Vector search
│   ├── deviations.json      # Deviation log
│   └── epics/               # Epic definitions
└── checkpoints/             # Recovery points
```

## Documentation

- [Commands Reference](COMMANDS.md)
- [Creating Specialists](CREATING-SPECIALISTS.md)
- [Pattern Library](PATTERN-LIBRARY.md)
- [Architecture](ARCHITECTURE.md)

## Example Session

```
> /orc plan "Build a blog API with authentication"

📋 Plan Created: Build a blog API with authentication

Epics (2):
  E1: User Authentication [3 features, 9 stories]
  E2: Blog CRUD API [4 features, 14 stories]

Total: 7 features, 23 stories

Run /orc approve to proceed

> /orc approve
✓ Plan approved

> /orc run
Starting execution...

[E1-F1-S1] Creating User model
  ├─ REASON: Loading pattern sp_bcrypt_001
  ├─ ACT: Creating src/models/User.ts
  ├─ VERIFY: ✓ Tests pass (4/4)
  └─ COMPLETE ✓

... (continues) ...

[EXECUTION COMPLETE]
  ├─ Stories: 23/23 completed
  ├─ Patterns learned: 5 new
  └─ Total time: 4m 32s
```
