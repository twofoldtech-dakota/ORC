---
description: "Show current state summary"
argument-hint: ""
allowed-tools: [Read]
---

# /orc:status Command

## Usage

```
/orc:status
```

## Description

Shows current ORC state summary including phase, progress, and health indicators.

## Output

```
📊 ORC Status

Session: a1b2c3d4-e5f6-7890-abcd-ef1234567890
Phase: execute
Started: 2025-01-30T10:00:00Z
Duration: 45m 32s

Goal: Build REST API with JWT authentication and user management

Progress:
  ┌─────────────────────────────────────────┐
  │ Epics     ████████░░░░░░░░░░░░  2/5     │
  │ Features  ████████████░░░░░░░░  6/10    │
  │ Stories   ████████████████░░░░  24/30   │
  └─────────────────────────────────────────┘

Current:
  Epic: E3 - Admin Dashboard
  Feature: E3-F1 - User Management UI
  Story: E3-F1-S2 - Create user list component

Health:
  ✓ Confidence: 94%
  ✓ Deviations: 2 (0 pending review)
  ⚠ Blocked: 1 story
  ✓ Patterns: 23 active

Recent Activity:
  [10:45:32] E3-F1-S1 completed
  [10:43:15] E3-F1 started
  [10:42:58] E2 completed (checkpoint saved)
  [10:30:12] Deviation recorded in E2-F3-S2

Files Modified (this session): 47
Tests Added: 32
Coverage: 82%

Commands:
  /orc:show          - View plan details
  /orc:show E3       - View current epic
  /orc:stop          - Pause execution
```

## Status Indicators

| Indicator | Meaning |
|-----------|---------|
| ✓ | Healthy/Good |
| ⚠ | Warning/Attention needed |
| ✗ | Error/Problem |

## Phase States

| Phase | Description |
|-------|-------------|
| `idle` | No active session |
| `plan` | Plan created, awaiting approval |
| `execute` | Executing stories |
| `execute (paused)` | Stopped mid-execution |
| `review` | Final review in progress |
| `learn` | Learning extraction in progress |
| `complete` | Session finished |

## Health Metrics

- **Confidence**: Overall confidence based on deviations and patterns
- **Deviations**: Count of approach deviations
- **Blocked**: Stories that failed after max attempts
- **Patterns**: Active patterns in learnings

## Error States

```
📊 ORC Status

Session: a1b2c3d4-e5f6-7890-abcd-ef1234567890
Phase: execute (ERROR)

⚠️  Issues Detected:

1. 3 stories blocked
   - E2-F2-S3: Database connection failed
   - E2-F2-S4: Depends on E2-F2-S3
   - E2-F3-S1: External API timeout

2. 2 deviations pending review
   - E2-F1-S2: Security-related change
   - E2-F1-S4: Architecture change

3. Confidence below threshold: 72%

Recommended Actions:
  /orc:retry E2-F2-S3     - Fix database connection
  /orc:show deviations    - Review pending deviations
  /orc:stop               - Pause for investigation
```

## No Session

```
📊 ORC Status

No active session.

Run /orc:plan <goal> to start a new session.
```
