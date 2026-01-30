---
description: "Execute approved epics"
argument-hint: "[<epic-id>]"
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash, Task]
---

# /orc run Command

## Usage

```
/orc run                # Execute all approved epics
/orc run <epic-id>      # Execute specific epic
```

## Description

Executes approved epics by invoking the Implementer and Validator agents for each story. Supports parallel execution of independent stories.

## Behavior

### Prerequisites
- Plan must exist
- At least one epic must be approved
- If specific epic: that epic must be approved

### Execution Flow

1. Verify prerequisites
2. Load state and plan
3. For each approved epic (priority order):
   a. For each feature:
      - Identify parallelizable stories
      - Execute stories (parallel where possible)
      - Validate each story
      - Handle failures with auto-fix loop
      - Save feature checkpoint on completion
   b. Save epic checkpoint on completion
4. Transition to Review phase
5. Run Learner

### Progress Output

```
Starting execution of 2 epics...

[E1-F1-S1] Creating User model with password hashing
  ├─ REASON: Loading pattern sp_bcrypt_001 (confidence: 94%)
  ├─ REASON: Analyzing src/models/ directory structure
  ├─ ACT: Creating src/models/User.ts
  ├─ ACT: Creating src/models/User.test.ts
  ├─ OBSERVE: Files created, checking types...
  ├─ VERIFY: ✓ Tests pass (4/4)
  ├─ VERIFY: ✓ Criteria met (4/4)
  └─ COMPLETE ✓ [8.2s]

[E1-F1-S2] Creating registration endpoint
  ├─ REASON: Reading User model for integration
  ├─ REASON: Loading pattern sp_api_001 (confidence: 89%)
  ├─ ACT: Creating src/routes/auth.ts
  ├─ ACT: Creating src/routes/auth.test.ts
  ├─ ACT: Modifying src/routes/index.ts
  ├─ OBSERVE: Files created, checking types...
  ├─ VERIFY: ✓ Tests pass (6/6)
  ├─ VERIFY: ✓ Criteria met (5/5)
  └─ COMPLETE ✓ [12.1s]

[FEATURE COMPLETE] E1-F1: User Registration
  ├─ Stories: 3/3 completed
  ├─ Deviations: 0
  ├─ Checkpoint saved: E1-F1-complete.json
  └─ Learning extraction triggered

... (continues for all features/epics) ...

[EPIC COMPLETE] E1: User Authentication
  ├─ Features: 3/3 completed
  ├─ Stories: 9/9 completed
  ├─ Deviations: 1 (reviewed: 0)
  ├─ Checkpoint saved: E1-complete.json
  └─ Starting E2...
```

### Failure Output

```
[E1-F2-S3] Creating password reset flow (attempt 2/3)
  ├─ PREVIOUS FAILURE: Missing email service dependency
  ├─ REASON: Adding email service mock for now...
  ├─ DEVIATION: Using mock email service instead of real integration
  │   └─ Reason: Email service not configured in project
  │   └─ Impact: low
  │   └─ Requires review: No
  ├─ ACT: Creating src/services/email.mock.ts
  ├─ ACT: Creating src/routes/password-reset.ts
  ├─ OBSERVE: Files created, checking types...
  ├─ VERIFY: ✓ Tests pass (5/5)
  ├─ VERIFY: ✓ Criteria met (4/4)
  └─ COMPLETE ✓ (with deviation) [15.3s]
```

### Blocked Story Output

```
[E1-F3-S2] Implementing email verification (attempt 3/3)
  ├─ PREVIOUS FAILURE: SMTP connection refused
  ├─ REASON: Trying alternative approach...
  ├─ ACT: Attempting local email service
  ├─ OBSERVE: Still failing - no SMTP server
  ├─ BLOCKED ✗
  │   └─ Reason: No email service available
  │   └─ Attempts: 3/3
  │   └─ Impact: Blocks E1-F3-S3
  └─ Continuing with non-dependent stories...
```

### Completion Output

```
[EXECUTION COMPLETE]
  ├─ Epics: 2/2 completed
  ├─ Features: 7/7 completed
  ├─ Stories: 23/23 completed
  ├─ Deviations: 2 (requires review: 1)
  ├─ Patterns learned: 5 new, 3 updated
  └─ Total time: 4m 32s

Run /orc show deviations to review pending deviations
```

### Partial Completion Output

```
[EXECUTION PAUSED]
  ├─ Epics: 1/2 completed
  ├─ Features: 5/7 completed
  ├─ Stories: 19/23 completed
  ├─ Blocked: 4 stories
  │   └─ E2-F2-S3: Database connection failed
  │   └─ E2-F2-S4: Depends on E2-F2-S3
  │   └─ E2-F3-S1: External API unavailable
  │   └─ E2-F3-S2: Depends on E2-F3-S1
  └─ Deviations: 3 (requires review: 2)

Run /orc retry E2-F2-S3 to retry blocked story
Run /orc show deviations to review pending deviations
```

## State Updates

- Sets `phase` to `execute`
- Updates `current_epic`, `current_feature`, `current_story`
- Updates `last_activity` continuously
- Updates story statuses as they complete
- Saves checkpoints at feature/epic boundaries
- Transitions to `review` phase on completion

## Parallel Execution

Stories without dependencies execute concurrently:

```
Feature E1-F1 with 4 stories:
  S1 (no deps)     ──────────►
  S2 (no deps)     ──────────►     (parallel)
  S3 (depends S1)       wait... ──────────►
  S4 (depends S2)       wait... ──────────►
```

## Error Handling

| Error | Response |
|-------|----------|
| No approved epics | "No approved epics. Run /orc approve first." |
| Invalid epic ID | "Epic '<id>' not found or not approved." |
| All stories blocked | "All remaining stories are blocked. Review blockers." |
| State corruption | "State file corrupted. Run /orc resume to recover." |
