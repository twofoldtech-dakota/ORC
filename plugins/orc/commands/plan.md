---
description: "Create or append epic to plan"
argument-hint: "<goal>"
allowed-tools: [Read, Write, Glob, Grep, Bash, Task]
---

# /orc plan Command

## Usage

```
/orc plan <goal>
```

## Description

Creates a new plan or appends to an existing plan based on the provided goal. The Planner agent decomposes the goal into a structured Epic→Feature→Story hierarchy.

## Behavior

### New Plan
If no plan exists:
1. Initialize `.orc/` directory structure
2. Run codebase analysis (if profile missing or stale)
   - Invoke Analyzer agent
   - Extract conventions, patterns, dependencies
   - Save codebase_profile.json
3. Invoke Planner agent with goal and codebase profile
4. Generate Epic→Feature→Story hierarchy aligned with codebase
5. Match patterns from learnings
6. Save plan to `.orc/plan/`
7. Display plan summary
8. Await approval

### Existing Plan
If plan already exists:
1. Prompt user: "Append or Replace?"
   - **Append**: Add new epic(s) to existing plan
   - **Replace**: Clear plan and create new
   - **Cancel**: Abort operation
2. Proceed based on choice

## Output Format

```
📋 Plan Created: <goal summary>

Epics (<count>):
  E1: <epic name> [<feature count> features, <story count> stories]
      ├─ F1: <feature name> (<story count> stories)
      ├─ F2: <feature name> (<story count> stories)
      └─ F3: <feature name> (<story count> stories)

  E2: <epic name> [<feature count> features, <story count> stories]
      ├─ F1: <feature name> (<story count> stories)
      └─ F2: <feature name> (<story count> stories)

Total: <total features> features, <total stories> stories
Estimated patterns matched: <count>

Run /orc show for full details
Run /orc approve to proceed
Run /orc show E1 for epic details
```

## Append Dialog

```
⚠️  Existing plan detected (<epic count> epics, <story count> stories)

[A]ppend as new epic
[R]eplace entire plan
[C]ancel

Choice:
```

## Directory Initialization

Creates if not exists:
```
.orc/
├── plan/
│   ├── state.json
│   ├── plan.json
│   ├── learnings.json
│   ├── embeddings.json
│   ├── deviations.json
│   ├── scratchpad.json
│   └── epics/
└── checkpoints/
```

## State Updates

- Sets `phase` to `plan`
- Creates new `session_id` (or preserves for append)
- Updates `goal` field
- Resets summaries for replace, increments for append

## Error Handling

| Error | Response |
|-------|----------|
| Empty goal | "Please provide a goal. Usage: /orc plan <goal>" |
| Planner timeout | "Planning took too long. Try a smaller goal." |
| Invalid learnings | "Warning: Could not load learnings, proceeding without patterns" |

## Example

```
> /orc plan "Build REST API with JWT authentication and user management"

🔍 Analyzing Codebase... ✓

  Project: typescript (express, prisma)
  Conventions: kebab-case files, camelCase functions
  Patterns: repository, zod validation, jwt auth
  Tests: jest (describe-it style)

📋 Plan Created: Build REST API with JWT authentication and user management

Epics (2):
  E1: User Authentication [3 features, 9 stories]
      ├─ F1: User Registration (3 stories)
      ├─ F2: User Login (3 stories)
      └─ F3: Password Reset (3 stories)

  E2: User Management [2 features, 6 stories]
      ├─ F1: Profile Management (3 stories)
      └─ F2: Account Settings (3 stories)

Total: 5 features, 15 stories
Estimated patterns matched: 6
Codebase alignment: 94%

Run /orc show for full details
Run /orc approve to proceed
```

### With Cached Profile

```
> /orc plan "Add email notifications"

🔍 Codebase Profile ✓ (cached, 2 hours ago)

📋 Plan Created: Add email notifications

Epics (1):
  E1: Email Notification System [2 features, 5 stories]
      ├─ F1: Email Service Integration (2 stories)
      └─ F2: Notification Templates (3 stories)

Total: 2 features, 5 stories
Codebase alignment: 96%

Run /orc approve to proceed
```
