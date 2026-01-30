---
name: orc
description: "Multi-Agent Orchestration System - invoke with /orc: commands for autonomous software development"
version: 1.0.0
triggers:
  - "/orc:"
---

# ORC - Multi-Agent Orchestration System

## Overview

ORC (Orchestrator) is an autonomous multi-agent orchestration system for software development. It transforms high-level goals into complete, tested implementations through a structured phased approach.

## Priority Order

**Quality > Capability > Developer Experience > Speed > Cost**

## Core Workflow

```
User Goal → Analyze Phase → Plan Phase → User Approval → Execute Phase → Review Phase → Learn Phase
```

## Commands

### Analysis Commands

| Command | Description |
|---------|-------------|
| `/orc:analyze` | Analyze codebase conventions and patterns |
| `/orc:analyze --force` | Force re-analysis (ignore cache) |
| `/orc:analyze --focus <areas>` | Analyze specific areas only |

### Planning Commands

| Command | Description |
|---------|-------------|
| `/orc:plan <goal>` | Create or append epic to plan |
| `/orc:show` | Display plan summary |
| `/orc:show <epic-id>` | Display epic details |
| `/orc:show deviations` | Show all deviations for review |
| `/orc:approve` | Approve all pending epics |
| `/orc:approve <epic-id>` | Approve specific epic |

### Execution Commands

| Command | Description |
|---------|-------------|
| `/orc:run` | Execute all approved epics |
| `/orc:run <epic-id>` | Execute specific epic |
| `/orc:next` | Execute next priority epic only |
| `/orc:stop` | Stop execution gracefully |
| `/orc:resume` | Resume from last checkpoint |
| `/orc:retry <story-id>` | Retry a blocked story |

### Learning & Utility Commands

| Command | Description |
|---------|-------------|
| `/orc:patterns` | Show learned patterns |
| `/orc:learn` | Force pattern extraction now |
| `/orc:status` | Show current state summary |
| `/orc:clear` | Clear plan and state |

## Command Routing

When the user invokes `/orc`, parse the subcommand and route accordingly:

1. **analyze** - Load `commands/analyze.md` and invoke Analyzer agent
2. **plan** - Load `commands/plan.md` and invoke Planner agent (auto-runs analyze if needed)
3. **show** - Load `commands/show.md` and display plan/epic/deviations
4. **approve** - Load `commands/approve.md` and update plan status
5. **run** - Load `commands/run.md` and invoke Orchestrator for execution
6. **next** - Load `commands/next.md` and execute single epic
7. **stop** - Load `commands/stop.md` and gracefully halt execution
8. **resume** - Load `commands/resume.md` and continue from checkpoint
9. **retry** - Load `commands/retry.md` and retry blocked story
10. **patterns** - Load `commands/patterns.md` and display patterns
11. **learn** - Load `commands/learn.md` and invoke Learner agent
12. **status** - Load `commands/status.md` and show state
13. **clear** - Load `commands/clear.md` and reset state

## Runtime Directory

ORC maintains state in `.orc/` directory:

```
.orc/
├── plan/
│   ├── state.json           # Current execution state
│   ├── plan.json            # Master plan metadata
│   ├── codebase_profile.json # Pre-flight codebase analysis
│   ├── learnings.json       # Accumulated patterns
│   ├── embeddings.json      # Vector embeddings
│   ├── deviations.json      # Deviation log
│   ├── scratchpad.json      # Working memory
│   └── epics/
│       ├── E1.json          # Epic definitions
│       └── ...
└── checkpoints/
    ├── E1-created.json      # Checkpoints
    └── ...
```

## Agent System

### Core Agents (7)

| Agent | Phase | Responsibility |
|-------|-------|----------------|
| **Orchestrator** | All | State management, phase control, contract enforcement |
| **Analyzer** | Analyze | Codebase conventions, patterns, dependency extraction |
| **Planner** | Plan | Goal decomposition into Epic→Feature→Story |
| **Implementer** | Execute | Code implementation with ReAct pattern |
| **Validator** | Execute | Test execution, acceptance criteria verification |
| **Reviewer** | Review | Final quality gate, deviation analysis |
| **Learner** | Learn | Pattern extraction, learnings persistence |

### Specialist Sub-Agents (12)

Spawned on-demand by core agents based on task requirements:

- **Architect** - System design, patterns, scalability
- **Product Designer** - UX flows, acceptance criteria
- **DevOps Engineer** - CI/CD, Docker, K8s, infrastructure
- **Full Stack Dev** - End-to-end implementation
- **Security Engineer** - Auth, encryption, OWASP
- **Database Engineer** - Schema design, migrations
- **Frontend Specialist** - React/Vue, CSS, accessibility
- **Backend Specialist** - REST, GraphQL, API design
- **QA Engineer** - Test strategy, edge cases
- **Biz Analyst** - Requirements, business logic
- **Content Strategist** - SEO, messaging, copy
- **Design Researcher** - UI reference research (frontend stories)

## Plan Hierarchy

```
Plan
└── Epic (self-contained project milestone)
    └── Feature (self-contained capability)
        └── Story (atomic task with acceptance criteria)
```

## Quality Gates

### Story Completion
- All acceptance criteria verified with evidence
- All unit tests pass
- Type checking passes (`tsc --noEmit`)
- No lint errors
- No new security vulnerabilities
- Approach compliance verified

### Feature Completion
- All stories completed or explicitly blocked
- Integration tests pass
- Feature-level acceptance criteria verified

### Epic Completion
- All features completed
- E2E tests pass (if applicable)
- Final review completed
- Full security scan passes

## Deviation Handling

Any deviation from `suggested_approach` must be:
1. Documented with reasoning
2. Flagged for user review (if security-related)
3. Tracked in deviation history
4. Factored into confidence scoring

Auto-require review for:
- Security-related deviations
- Architecture changes
- High impact deviations

## Auto-Fix Loop

On story failure:
1. Retry with previous failure context (up to 3 attempts)
2. Try alternative patterns if available
3. If still failing, mark as BLOCKED
4. Continue with non-dependent stories
5. Notify user of blocked items

## Parallel Execution

Independent stories execute in parallel:
- Dependencies analyzed at plan time
- No conflicts possible (enforced by Planner)
- Parallel stories run concurrently
