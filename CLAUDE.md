# ORC - Orchestrator

ORC is a multi-agent orchestration system for autonomous software development. It coordinates specialized AI agents through a structured Epic → Feature → Story hierarchy with quality gates at each level.

## Architecture

### Core Agents (7)
- **Orchestrator** - Top-level coordinator, manages execution flow
- **Planner** - Decomposes goals into Epic/Feature/Story hierarchy
- **Analyzer** - Pre-flight codebase analysis (ANALYZE phase)
- **Implementer** - Executes story implementations (ReAct pattern)
- **Validator** - Verifies implementations against acceptance criteria
- **Reviewer** - Final quality gates, Reflexion pattern for learnings
- **Learner** - Extracts and persists insights for future use

### Specialist Agents (12)
- **Architect** - System design, architectural decisions
- **Security** - Auth, encryption, vulnerability scanning
- **Database** - Schema design, migrations, queries
- **Frontend** - React/Vue, CSS, accessibility, animations
- **Backend** - APIs, server logic, integrations
- **Fullstack** - End-to-end implementations
- **DevOps** - CI/CD, Docker, infrastructure
- **QA** - Test scenarios, edge cases, coverage
- **Product Designer** - UX patterns, user flows
- **Content Strategist** - Copy, messaging, tone
- **Biz Analyst** - Requirements, acceptance criteria
- **Design Researcher** - UI reference research (frontend stories)

## Key Directories

```
.claude-plugin/        # Marketplace catalog
  marketplace.json     # Lists available plugins
plugins/orc/           # ORC plugin package
  .claude-plugin/      # Plugin manifest
    plugin.json
  skills/orc/          # Skill definition (/orc commands)
  agents/              # Agent definitions (19 files)
  commands/            # Command definitions (13 files)
  hooks/               # Lifecycle hooks (hooks.json)
  contracts/           # JSON schemas for data contracts
  patterns/            # Reusable implementation patterns
    frontend/          # Frontend-specific patterns
      _references/     # Linear, Vercel, Stripe, Remotion patterns
      components/      # Button, card, form patterns
      interactions/    # Hover, transitions, animations
  docs/                # Documentation
.orc/                  # Runtime state (created during execution)
  design/              # Design research & innovation assessments
```

## Frontend Quality System

ORC includes a comprehensive frontend quality system to prevent generic "AI slop" output.

### Design Research Phase
For stories tagged `frontend`, `ui`, `component`, `page`, or `layout`:
1. Implementer spawns Design Researcher agent
2. Research document created at `.orc/design/research/{story-id}.md`
3. Implementation proceeds using research as guide

### Quality Gates (Frontend Stories)

**Validator checks:**
- Anti-slop detection (layout, styling, interaction, animation patterns)
- Interaction requirements (all states: hover, focus, active, loading, disabled)
- Design system compliance (spacing, typography, colors, animation)

**Reviewer checks:**
- Design review gate (10-point review)
- Innovation scoring (minimums by component type)

### Anti-Slop Rules
Blocks generic patterns like:
- Default Tailwind colors (blue-500, gray-100)
- `rounded-lg` on everything
- `transition-all duration-300`
- Hover effects that only change opacity
- No focus states on interactive elements

### Innovation Minimums
- Landing pages: 7/10
- Marketing components: 8/10
- Documentation: 6/10
- Dashboard UI: 5/10
- Forms: 4/10

## Configuration

`master-config.json` contains:
- `anti_slop_rules` - Patterns to detect and block
- `design_review_gate` - 10-point design review criteria
- `innovation_minimums` - Required scores by component type

## Contracts

Key schemas in `plugins/orc/contracts/`:
- `story.schema.json` - Story definition with acceptance criteria
- `design-system.schema.json` - Design system constraints
- `interaction-requirements.schema.json` - Required interaction states
- `innovation-assessment.schema.json` - Innovation scoring structure

## Reference Standards

Frontend implementations should match quality of:
- **Linear.app** - Spring animations, keyboard-first, dark mode
- **Remotion.dev** - Documentation UX, developer delight
- **Vercel.com** - Information hierarchy, dark mode excellence
- **Stripe.com** - Pixel-perfect alignment, dashboard patterns

## Working with ORC

### Adding a Frontend Story
1. Tag story with `frontend`, `ui`, `component`, `page`, or `layout`
2. Design Researcher will auto-spawn
3. Follow research document recommendations
4. Implement all required interaction states
5. Use design system values (no arbitrary values)
6. Document innovation techniques

### Extending Patterns
Add new patterns to `plugins/orc/patterns/frontend/`:
- Components: `components/{category}/{pattern-name}.md`
- Interactions: `interactions/{pattern-name}.md`
- References: `_references/{source}-patterns.md`

### Modifying Quality Rules
Edit `master-config.json`:
- Add slop patterns to detect
- Adjust innovation minimums
- Update design review criteria
