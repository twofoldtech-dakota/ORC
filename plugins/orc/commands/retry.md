---
description: "Retry a blocked story"
argument-hint: "<story-id>"
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash, Task]
---

# /orc:retry Command

## Usage

```
/orc:retry <story-id>
```

## Description

Retries a blocked story with fresh context. Resets attempt counter and allows the story to be re-executed.

## Behavior

1. Verify story exists and is blocked
2. Reset story attempt counter
3. Clear previous failure context
4. Re-execute story
5. Resume normal execution if successful

## Output

```
🔄 Retrying E2-F2-S3: Database migration setup

Previous attempts: 3
Previous failure: Database connection refused

Resetting attempt counter...

[E2-F2-S3] Database migration setup (attempt 1/3)
  ├─ REASON: Fresh attempt with clean context
  ├─ REASON: Checking database connectivity...
  ├─ OBSERVE: Database connection successful
  ├─ ACT: Creating migration files
  ├─ OBSERVE: Files created, running migration...
  ├─ VERIFY: ✓ Migration applied successfully
  ├─ VERIFY: ✓ Tests pass (4/4)
  └─ COMPLETE ✓ [9.8s]

✓ Story unblocked

Previously blocked dependents now ready:
  - E2-F2-S4: Create seed data

Continue execution? [Y/n]:
```

## Prerequisites

- Story must exist
- Story must be in `blocked` status
- Plan must be in `execute` phase

## State Updates

- Resets story `attempts` to 0
- Changes story `status` from `blocked` to `pending`
- Clears `result.failure_reason`
- Updates `blocked_stories` in learnings

## Use Cases

- After fixing external dependency (database, API, etc.)
- After environment configuration change
- After manual code fix
- When ready to retry after investigation

## Error Handling

| Error | Response |
|-------|----------|
| Story not found | "Story '<id>' not found." |
| Not blocked | "Story '<id>' is not blocked. Status: <status>" |
| Not in execute phase | "Cannot retry during <phase> phase." |

## Example Flow

```
# Story blocks during execution
[E2-F2-S3] Database migration setup (attempt 3/3)
  └─ BLOCKED ✗ - Database connection refused

# User fixes database connection
$ docker-compose up -d postgres

# User retries the story
> /orc:retry E2-F2-S3

# Story succeeds
[E2-F2-S3] Database migration setup (attempt 1/3)
  └─ COMPLETE ✓

# Execution continues automatically
```
