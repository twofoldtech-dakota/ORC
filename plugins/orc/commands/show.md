---
description: "Display plan summary, epic details, or deviations"
argument-hint: "[<epic-id>] [deviations]"
allowed-tools: [Read]
---

# /orc:show Command

## Usage

```
/orc:show                    # Show plan summary
/orc:show <epic-id>          # Show epic details
/orc:show deviations         # Show all deviations
```

## Description

Displays plan information, epic details, or deviation log based on arguments.

## Behaviors

### Plan Summary (`/orc:show`)

Shows overview of entire plan:

```
📋 Plan: <goal>
Status: <phase> | Session: <session_id>
Created: <timestamp>

Epics:
  ✓ E1: User Authentication [COMPLETED]
     └─ 3/3 features, 9/9 stories

  ▶ E2: API Endpoints [IN_PROGRESS]
     └─ 1/4 features, 5/12 stories
     └─ Current: E2-F2-S3 - Create post endpoint

  ○ E3: Admin Dashboard [PENDING]
     └─ 2 features, 8 stories

Progress:
  Epics:    1/3 completed
  Features: 4/9 completed
  Stories:  14/29 completed (3 blocked)

Confidence: 94%
Deviations: 2 (1 pending review)

Commands:
  /orc:show E2          - View epic details
  /orc:show deviations  - Review deviations
  /orc:run              - Continue execution
```

### Epic Details (`/orc:show <epic-id>`)

Shows detailed view of specific epic:

```
📋 Epic E1: User Authentication
Status: COMPLETED
Priority: 1

Description:
Complete authentication system with registration, login, and password reset.

Features:
  ✓ F1: User Registration [COMPLETED]
     ├─ ✓ S1: Create User model with password hashing
     ├─ ✓ S2: Create registration endpoint
     └─ ✓ S3: Add email validation

  ✓ F2: User Login [COMPLETED]
     ├─ ✓ S1: Create login endpoint
     ├─ ✓ S2: Implement JWT token generation
     └─ ✓ S3: Add refresh token logic

  ✓ F3: Password Reset [COMPLETED]
     ├─ ✓ S1: Create password reset request endpoint
     ├─ ✓ S2: Create password reset confirmation endpoint
     └─ ✓ S3: Add email notification

Patterns Used: 4
  - sp_bcrypt_001 (password hashing)
  - sp_jwt_001 (token generation)
  - sp_email_001 (email service)
  - sp_validation_001 (input validation)

Deviations: 1
  - S2: Used RS256 instead of HS256 for JWT (approved)

Completed: 2025-01-30T12:30:00Z
Duration: 15m 42s
```

### Story Details (when showing epic)

For in-progress or pending stories, show additional detail:

```
  ▶ S3: Create post endpoint [IN_PROGRESS] (attempt 2/3)
     │  Description: RESTful endpoint for creating blog posts
     │  Acceptance Criteria:
     │    ✓ POST /api/posts creates new post
     │    ○ Request validated with Zod
     │    ○ Returns 201 with created post
     │  Files: src/routes/posts.ts, src/routes/posts.test.ts
     │  Suggested: Use controller-service pattern (sp_api_001)
     │  Previous failure: Validation schema import error
     └─
```

### Deviations (`/orc:show deviations`)

Shows all deviations requiring review:

```
⚠️  Deviations Pending Review

1. [E1-F2-S2] JWT Algorithm Change
   Suggested: HS256 symmetric algorithm
   Actual: RS256 asymmetric algorithm
   Reason: Project security policy requires asymmetric signing
   Category: security
   Impact: medium
   Status: PENDING REVIEW

2. [E2-F1-S3] Database Library
   Suggested: TypeORM
   Actual: Prisma
   Reason: Project already uses Prisma for other models
   Category: library_choice
   Impact: low
   Status: PENDING REVIEW

Reviewed (2):
  ✓ [E1-F1-S1] Bcrypt cost factor: 12 instead of 10 (approved)
  ✓ [E1-F3-S2] Mock email service used (approved)

Commands:
  /orc:approve deviation 1   - Approve specific deviation
  /orc:approve deviations    - Approve all pending
```

## Status Icons

| Icon | Meaning |
|------|---------|
| ✓ | Completed |
| ▶ | In Progress |
| ○ | Pending |
| ✗ | Blocked |
| ⚠ | Needs Review |

## Error Handling

| Error | Response |
|-------|----------|
| No plan exists | "No plan found. Run /orc:plan <goal> to create one." |
| Invalid epic ID | "Epic '<id>' not found. Available epics: E1, E2, E3" |
| Invalid argument | "Unknown argument. Usage: /orc:show [epic-id|deviations]" |
