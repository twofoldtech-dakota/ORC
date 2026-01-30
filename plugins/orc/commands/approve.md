---
description: "Approve pending epics or deviations"
argument-hint: "[<epic-id>] [deviation <id>] [deviations]"
allowed-tools: [Read, Write]
---

# /orc:approve Command

## Usage

```
/orc:approve                    # Approve all pending epics
/orc:approve <epic-id>          # Approve specific epic
/orc:approve deviation <id>     # Approve specific deviation
/orc:approve deviations         # Approve all deviations
```

## Description

Approves plan items or deviations for execution. Approval is **mandatory** before any execution can begin.

## Behaviors

### Approve All Epics (`/orc:approve`)

Approves all pending epics in the plan:

```
✓ Plan approved

Approved Epics:
  E1: User Authentication [3 features, 9 stories]
  E2: API Endpoints [4 features, 12 stories]

Total: 7 features, 21 stories ready for execution

Run /orc:run to execute all epics
Run /orc:next to execute next epic only
```

### Approve Specific Epic (`/orc:approve <epic-id>`)

Approves single epic:

```
✓ Epic E1 approved: User Authentication

Features: 3
Stories: 9
Patterns matched: 4

Run /orc:run E1 to execute this epic
Run /orc:approve E2 to approve next epic
```

### Approve Deviation (`/orc:approve deviation <id>`)

Approves a specific deviation:

```
✓ Deviation approved

[E1-F2-S2] JWT Algorithm Change
  Suggested: HS256 symmetric algorithm
  Actual: RS256 asymmetric algorithm
  Reason: Project security policy requires asymmetric signing
  Status: APPROVED

Remaining deviations pending review: 1
```

### Approve All Deviations (`/orc:approve deviations`)

Approves all pending deviations:

```
✓ All deviations approved (2 total)

1. [E1-F2-S2] JWT Algorithm Change - APPROVED
2. [E2-F1-S3] Database Library - APPROVED

No deviations pending review.
Confidence restored to 100%
```

## State Updates

### Epic Approval
- Sets epic `status` to `approved`
- Sets epic `approved_at` timestamp
- Updates `epics_summary.approved` count

### Deviation Approval
- Sets deviation `reviewed` to `true`
- Sets deviation `approved` to `true`
- Removes confidence penalty
- Updates `deviations_pending_review` count

## Validation

### Before Approving Epics
- Plan must exist
- Plan must be in `plan` phase
- Epic must be in `pending` status

### Before Approving Deviations
- Deviation must exist
- Deviation must not already be reviewed

## Error Handling

| Error | Response |
|-------|----------|
| No plan | "No plan found. Run /orc:plan <goal> first." |
| Already approved | "Epic E1 is already approved." |
| Invalid epic ID | "Epic '<id>' not found. Available: E1, E2" |
| Invalid deviation ID | "Deviation '<id>' not found." |
| No pending items | "Nothing to approve. All items already approved." |
| Blocked epic | "Cannot approve E2: depends on blocked epic E1." |

## Approval Flow

```
/orc:plan "Build API"
    │
    ▼
Plan created (pending approval)
    │
    ▼
/orc:show           # Review plan
    │
    ▼
/orc:approve        # Approve all
    │         or
/orc:approve E1     # Approve specific
    │
    ▼
Ready for execution
    │
    ▼
/orc:run
```

## Confirmation Output

When approving potentially risky items:

```
⚠️  Security Deviation Review

[E1-F2-S2] JWT Algorithm Change
  This deviation involves security-related changes.

  Suggested: HS256 symmetric algorithm
  Actual: RS256 asymmetric algorithm
  Reason: Project security policy requires asymmetric signing

  Impact Assessment:
  - RS256 is generally more secure for distributed systems
  - Requires public/private key management
  - No functional regression expected

Approve this deviation? [y/N]:
```
