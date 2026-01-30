<div align="center">

![Claude Code Plugin](https://img.shields.io/badge/Claude_Code-Plugin-7C3AED?style=for-the-badge&logo=anthropic&logoColor=white)
![Version](https://img.shields.io/badge/version-1.0.0-blue?style=for-the-badge)
![License](https://img.shields.io/badge/license-MIT-green?style=for-the-badge)

# ORC

### Autonomous Software Development Through Multi-Agent Orchestration

**One prompt. Nineteen agents. Production-ready code.**

```bash
/plugin marketplace add twofoldtech-dakota/ORC
```

[Installation](#-installation) • [Quick Start](#-quick-start) • [Commands](#-commands) • [Architecture](#-architecture) • [Documentation](#-documentation)

</div>

---

## Overview

ORC transforms high-level goals into complete, tested implementations. You describe what you want; ORC handles decomposition, implementation, validation, and learning—autonomously.

```
"Build a user authentication system with email verification"
                              ↓
        ┌─────────────────────────────────────────────────────────┐
        │  ANALYZE → PLAN → EXECUTE → VALIDATE → REVIEW → LEARN  │
        └─────────────────────────────────────────────────────────┘
                              ↓
           ✓ 3 Epics, 9 Features, 27 Stories completed
           ✓ 94 tests passing
           ✓ Patterns extracted for future use
```

---

## 📦 Installation

### Prerequisites

[Claude Code CLI](https://docs.anthropic.com/claude-code) installed and authenticated.

```bash
# Verify Claude Code is installed
claude --version
```

### Install from Marketplace

Start Claude Code and run these commands:

```bash
# Add the ORC marketplace
/plugin marketplace add twofoldtech-dakota/ORC

# Install the plugin
/plugin install orc@orc-marketplace
```

Or use interactive discovery:
```bash
/plugin discover
```

### Verify Setup

```bash
/orc:status
```

Expected output:
```
📊 ORC Status
No active session.
Run /orc:plan <goal> to start.
```

### Updating & Uninstalling

```bash
# Update the marketplace
/plugin marketplace update orc-marketplace

# Uninstall
/plugin uninstall orc@orc-marketplace
```

---

## 🚀 Quick Start

### Basic Usage

Navigate to your project and invoke ORC with a goal:

```bash
cd your-project

# Start Claude Code
claude

# Plan and execute a feature
> /orc:plan "Add user authentication with JWT tokens"
```

ORC will:
1. **Analyze** your codebase (conventions, patterns, dependencies)
2. **Plan** an Epic → Feature → Story hierarchy
3. **Present** the plan for your approval
4. **Execute** each story with specialist agents
5. **Validate** against acceptance criteria
6. **Learn** patterns for future executions

### Example Session

```
You: /orc:plan "Build a REST API with user management"

ORC: 🔍 Analyzing Codebase... ✓
     Project: typescript (express, prisma)
     Patterns: repository, zod validation

     📋 Plan Created: Build a REST API with user management

     Epics (2):
       E1: User Authentication [3 features, 9 stories]
           ├─ F1: User Registration (3 stories)
           ├─ F2: User Login (3 stories)
           └─ F3: Password Reset (3 stories)

       E2: User Management [2 features, 6 stories]
           ├─ F1: Profile Management (3 stories)
           └─ F2: Account Settings (3 stories)

     Total: 5 features, 15 stories
     Codebase alignment: 94%

     Run /orc:approve to proceed

You: /orc:approve

ORC: ✓ Plan approved. Run /orc:run to execute.

You: /orc:run

ORC: 🚀 Starting execution...

     [E1-F1-S1] Creating User model with password hashing
       ├─ REASON: Loading pattern sp_bcrypt_001
       ├─ ACT: Creating src/models/User.ts
       ├─ ACT: Creating src/models/User.test.ts
       ├─ VERIFY: ✓ Tests pass (4/4)
       └─ COMPLETE ✓ [8.2s]

     ... (continues autonomously) ...

     [EXECUTION COMPLETE]
       ├─ Stories: 15/15 completed
       ├─ Tests: 47 passing
       └─ Patterns learned: 3 new
```

---

## 📋 Commands

| Command | Description | Example |
|---------|-------------|---------|
| `/orc:analyze` | Profile codebase conventions and patterns | `/orc:analyze --focus security` |
| `/orc:plan <goal>` | Create Epic → Feature → Story hierarchy | `/orc:plan "Add payments"` |
| `/orc:show` | Display plan summary | `/orc:show E1` |
| `/orc:approve` | Approve plan for execution | `/orc:approve E1` |
| `/orc:run` | Execute approved epics | `/orc:run E1` |
| `/orc:next` | Execute next epic only | `/orc:next` |
| `/orc:stop` | Gracefully pause execution | `/orc:stop` |
| `/orc:resume` | Continue from checkpoint | `/orc:resume` |
| `/orc:retry <id>` | Retry a blocked story | `/orc:retry E1-F2-S3` |
| `/orc:patterns` | Show learned patterns | `/orc:patterns auth` |
| `/orc:learn` | Force pattern extraction | `/orc:learn` |
| `/orc:status` | Show current state | `/orc:status` |
| `/orc:clear` | Reset plan and state | `/orc:clear` |

---

## 🔄 Workflow

<!-- [WORKFLOW DIAGRAM PLACEHOLDER] -->

```
┌──────────────────────────────────────────────────────────────────────────┐
│                              USER GOAL                                    │
│                    "Build a dashboard with analytics"                     │
└─────────────────────────────────┬────────────────────────────────────────┘
                                  │
                                  ▼
┌──────────────────────────────────────────────────────────────────────────┐
│                           ANALYZE PHASE                                   │
│  ┌─────────────┐                                                         │
│  │  ANALYZER   │ → Codebase Profile → Conventions, Patterns, Stack       │
│  └─────────────┘                                                         │
└─────────────────────────────────┬────────────────────────────────────────┘
                                  │
                                  ▼
┌──────────────────────────────────────────────────────────────────────────┐
│                            PLAN PHASE                                     │
│  ┌─────────────┐                                                         │
│  │   PLANNER   │ → Epic → Feature → Story decomposition                  │
│  └──────┬──────┘   + Pattern matching from learnings                     │
│         │                                                                 │
│         ├──→ Architect (system design)                                   │
│         ├──→ Product Designer (UX flows)                                 │
│         └──→ Biz Analyst (requirements)                                  │
└─────────────────────────────────┬────────────────────────────────────────┘
                                  │
                                  ▼
                         [ USER APPROVAL ]
                                  │
                                  ▼
┌──────────────────────────────────────────────────────────────────────────┐
│                          EXECUTE PHASE                                    │
│  ┌─────────────┐      ┌─────────────┐      ┌─────────────┐              │
│  │ IMPLEMENTER │ ───▶ │  VALIDATOR  │ ───▶ │  REVIEWER   │              │
│  └──────┬──────┘      └─────────────┘      └─────────────┘              │
│         │               Tests, Types,        Quality Gates,              │
│         │               Lint, Security       Design Review               │
│         │                                                                 │
│         ├──→ Frontend Specialist (React, CSS, a11y)                      │
│         ├──→ Backend Specialist (APIs, services)                         │
│         ├──→ Database Engineer (schema, migrations)                      │
│         ├──→ Security Engineer (auth, encryption)                        │
│         └──→ DevOps Engineer (CI/CD, Docker)                             │
└─────────────────────────────────┬────────────────────────────────────────┘
                                  │
                                  ▼
┌──────────────────────────────────────────────────────────────────────────┐
│                           LEARN PHASE                                     │
│  ┌─────────────┐                                                         │
│  │   LEARNER   │ → Extract patterns, anti-patterns, preferences          │
│  └─────────────┘   → Update embeddings for future matching               │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## 🏗 Architecture

### Agent Hierarchy

ORC deploys **19 specialized agents** organized into two tiers:

#### Core Agents (7)

| Agent | Role | Pattern |
|-------|------|---------|
| **Orchestrator** | State management, phase control, delegation | State Machine |
| **Analyzer** | Codebase profiling, convention extraction | Pre-flight Analysis |
| **Planner** | Goal decomposition, dependency mapping | Tree-of-Thoughts |
| **Implementer** | Story execution, code generation | ReAct |
| **Validator** | Test execution, criteria verification | Chain-of-Verification |
| **Reviewer** | Quality gates, deviation analysis | Reflexion |
| **Learner** | Pattern extraction, knowledge persistence | Memory Persistence |

#### Specialist Agents (12)

| Specialist | Domain | Spawned By |
|------------|--------|------------|
| **Architect** | System design, patterns, scalability | Planner, Implementer |
| **Security** | Auth, encryption, OWASP compliance | Implementer, Reviewer |
| **Database** | Schema design, migrations, optimization | Planner, Implementer |
| **Frontend** | React/Vue, CSS, accessibility, animations | Implementer |
| **Backend** | REST, GraphQL, business logic | Implementer |
| **Fullstack** | End-to-end feature implementation | Implementer |
| **DevOps** | CI/CD, Docker, Kubernetes | Planner, Implementer |
| **QA** | Test strategy, edge cases, coverage | Validator, Reviewer |
| **Product Designer** | UX flows, acceptance criteria | Planner |
| **Biz Analyst** | Requirements, business rules | Planner |
| **Content Strategist** | SEO, messaging, copy | Planner, Implementer |
| **Design Researcher** | UI reference research, innovation | Implementer |

<!-- [ARCHITECTURE DIAGRAM PLACEHOLDER] -->

### Plan Hierarchy

```
Plan
└── Epic (self-contained project milestone)
    └── Feature (self-contained capability)
        └── Story (atomic task with acceptance criteria)
```

**Constraints enforced:**
- No cross-epic dependencies
- No cross-feature dependencies within epic
- Stories can only depend on stories within same feature

### Quality Gates

| Level | Gate | Checks |
|-------|------|--------|
| **Story** | Validator | Tests pass, types check, lint clean, criteria met |
| **Feature** | Validator | All stories complete, integration tests pass |
| **Epic** | Reviewer | Security scan, design review, innovation score |

### Checkpoint System

ORC saves checkpoints at every boundary:

```
.orc/
├── plan/
│   ├── state.json              # Current execution state
│   ├── plan.json               # Master plan metadata
│   ├── codebase_profile.json   # Analysis results
│   ├── learnings.json          # Accumulated patterns
│   └── epics/
│       ├── E1.json
│       └── E2.json
└── checkpoints/
    ├── E1-F1-complete.json
    ├── E1-complete.json
    └── ...
```

Execution can resume from any checkpoint after interruption.

---

## 🎨 Frontend Quality System

ORC includes an anti-slop system that **blocks generic AI output**.

### Anti-Slop Detection

| Category | Blocked Patterns |
|----------|-----------------|
| **Layout** | Generic hero sections, uniform card grids, 4-column footers |
| **Styling** | Default Tailwind colors (`blue-500`), `rounded-lg` everywhere |
| **Interaction** | Opacity-only hovers, missing focus states |
| **Animation** | `transition-all duration-300`, linear easing, no exit animations |

### Innovation Scoring

Every frontend component receives an innovation score. **Below minimum = blocked.**

| Component Type | Minimum Score |
|----------------|---------------|
| Landing Page | 7/10 |
| Marketing Component | 8/10 |
| Dashboard UI | 5/10 |
| Forms | 4/10 |

### Design Research Phase

For frontend stories, ORC automatically:
1. Spawns a **Design Researcher** agent
2. Analyzes world-class references (Linear, Vercel, Stripe, Remotion)
3. Documents techniques and anti-patterns
4. Guides implementation with research insights

---

## 🧠 Learning System

ORC gets smarter with every execution.

### Pattern Types

```json
{
  "success_patterns": [],      // What works
  "anti_patterns": [],         // What doesn't work
  "user_preferences": {},      // How you like things
  "project_knowledge": {},     // Project-specific info
  "technique_stats": {}        // Success/failure rates
}
```

### Pattern Matching

1. Story description generates semantic embedding
2. Embeddings compared against learned patterns
3. Matches above 80% similarity attached as `suggested_approach`
4. Patterns ranked by confidence score

### Confidence Decay

```
confidence = base_confidence × recency_factor × success_factor
```

- Unused patterns decay over time
- Anti-patterns never decay
- High-success patterns maintain confidence

---

## 📁 Project Structure

```
ORC/
├── .claude-plugin/
│   └── marketplace.json         # Marketplace catalog
├── plugins/
│   └── orc/                     # ORC plugin
│       ├── .claude-plugin/
│       │   └── plugin.json      # Plugin manifest
│       ├── skills/orc/SKILL.md  # Skill definition
│       ├── agents/              # Agent definitions (19 files)
│       ├── commands/            # Command definitions (13 files)
│       ├── hooks/hooks.json     # Lifecycle hooks
│       ├── contracts/           # JSON Schema definitions
│       ├── patterns/            # Implementation patterns
│       │   ├── frontend/
│       │   ├── api/
│       │   ├── auth/
│       │   └── database/
│       └── docs/                # Documentation
│
└── master-config.json           # Quality rules & thresholds
```

---

## ⚙️ Configuration

### `master-config.json`

```json
{
  "anti_slop_rules": {
    "enabled": true,
    "on_detection": {
      "action": "block_completion",
      "message": "Slop pattern detected. Research better approaches."
    }
  },
  "innovation_minimums": {
    "landing_page": 7,
    "marketing_component": 8,
    "dashboard_ui": 5,
    "forms": 4
  },
  "design_review_gate": {
    "enabled": true,
    "max_revision_rounds": 2
  }
}
```

---

## 📚 Documentation

| Document | Description |
|----------|-------------|
| [Architecture](plugins/orc/docs/ARCHITECTURE.md) | Deep dive into agent system |
| [Commands](plugins/orc/docs/COMMANDS.md) | Complete command reference |
| [Pattern Library](plugins/orc/docs/PATTERN-LIBRARY.md) | Available implementation patterns |
| [Creating Specialists](plugins/orc/docs/CREATING-SPECIALISTS.md) | Guide to adding new agents |

---

## 🤝 Contributing

### Adding Patterns

```bash
# Create new pattern
plugins/orc/patterns/{category}/{pattern-name}.md

# Include:
# - Reference sources
# - All states (hover, focus, active, etc.)
# - Code examples
# - Anti-patterns to avoid
```

### Adding Specialists

```bash
# Create new specialist
plugins/orc/agents/{specialist-name}.md

# Required frontmatter:
# ---
# name: specialist-name
# type: specialist
# model: opus
# tools: [Read, Write, ...]
# spawned_by: [implementer]
# ---
```

---

## 📄 License

MIT License — see [LICENSE](LICENSE) for details.

---

<div align="center">

**Built for developers who ship.**

*One prompt. Complete implementation.*

[Report Bug](https://github.com/twofoldtech-dakota/ORC/issues) • [Request Feature](https://github.com/twofoldtech-dakota/ORC/issues) • [Discussions](https://github.com/twofoldtech-dakota/ORC/discussions)

</div>
