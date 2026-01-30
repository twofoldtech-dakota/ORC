---
description: "Execute next priority epic only"
argument-hint: ""
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash, Task]
---

# /orc next Command

## Usage

```
/orc next
```

## Description

Executes only the next priority epic, then pauses. Useful for incremental execution with review between epics.

## Behavior

1. Identify next approved epic by priority
2. Execute all features/stories in that epic
3. Save checkpoint
4. Trigger learning extraction
5. Pause and show summary

## Output

```
Starting execution of E1: User Authentication...

[E1-F1-S1] Creating User model with password hashing
  ├─ REASON: Loading pattern sp_bcrypt_001 (confidence: 94%)
  ...
  └─ COMPLETE ✓ [8.2s]

... (all stories in epic) ...

[EPIC COMPLETE] E1: User Authentication
  ├─ Features: 3/3 completed
  ├─ Stories: 9/9 completed
  ├─ Deviations: 1 (reviewed: 0)
  ├─ Checkpoint saved: E1-complete.json
  └─ Patterns learned: 3 new, 1 updated

Execution paused.

Next epic: E2 - API Endpoints [4 features, 12 stories]

Commands:
  /orc next        - Execute next epic
  /orc run         - Execute all remaining epics
  /orc show E2     - Preview next epic
```

## Use Cases

- Step-by-step execution with human review
- Large projects requiring incremental validation
- When user wants to review output between epics
- Learning and observing system behavior

## State Updates

- Same as `/orc run` but pauses after epic completion
- Sets `phase` to `execute` (paused at epic boundary)
- Does not transition to `review` until all epics done

## Error Handling

| Error | Response |
|-------|----------|
| No next epic | "All approved epics completed. Run /orc show for summary." |
| No approved epics | "No approved epics. Run /orc approve first." |
| Current epic blocked | "Current epic is blocked. Run /orc show to see blockers." |
