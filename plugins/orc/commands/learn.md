---
description: "Force pattern extraction now"
argument-hint: ""
allowed-tools: [Read, Write, Glob, Grep]
---

# /orc:learn Command

## Usage

```
/orc:learn
```

## Description

Forces immediate pattern extraction from the current session. Normally learning happens automatically after feature completion, but this command triggers it manually.

## Behavior

1. Invoke Learner agent with current session data
2. Extract patterns from completed stories
3. Update learnings database
4. Generate/update embeddings
5. Display extraction summary

## Output

```
🧠 Learning extraction started...

Analyzing completed stories...
  ├─ E1: 9 stories completed
  ├─ E2: 6 stories completed (6 pending)
  └─ Total: 15 completed stories

Extracting patterns...

New Patterns (3):
  ✓ sp_zod_validation_001
    Task: Request validation with Zod schemas
    Confidence: 95%
    Extracted from: E1-F1-S2, E1-F2-S1, E2-F1-S1

  ✓ sp_prisma_relations_001
    Task: Prisma model relations
    Confidence: 90%
    Extracted from: E1-F1-S1, E2-F1-S2

  ✓ sp_error_boundary_001
    Task: React error boundaries
    Confidence: 88%
    Extracted from: E2-F2-S3

Updated Patterns (5):
  ↑ sp_bcrypt_001: Confidence 92% → 94% (+2 successes)
  ↑ sp_jwt_001: Confidence 87% → 89% (+1 success)
  ↑ sp_rest_001: Confidence 89% → 91% (+2 successes)
  ↓ sp_email_001: Confidence 45% → 35% (recency decay)
  = sp_cors_001: Confidence 95% (no change)

New Anti-Patterns (1):
  ✗ ap_inline_styles_001
    Pattern: Inline styles in React components
    Why bad: Inconsistent styling, hard to maintain
    Alternative: Use CSS modules or styled-components
    Extracted from: E2-F2-S2 (deviation)

Embeddings Updated:
  ├─ New embeddings: 3
  ├─ Updated embeddings: 5
  └─ Model: text-embedding-3-small

User Preferences Detected:
  ├─ Preferred libraries: zod, prisma, bcrypt
  ├─ Coding style: 2-space indent, single quotes
  └─ Test framework: vitest

Learnings saved to .orc/plan/learnings.json
```

## Use Cases

- Force learning mid-session
- After significant manual changes
- Before starting new project (to capture patterns)
- Debugging pattern matching

## State Updates

- Updates `learnings.json`
- Updates `embeddings.json`
- Does not change execution state

## Error Handling

| Error | Response |
|-------|----------|
| No completed stories | "No completed stories to learn from." |
| Embedding error | "Warning: Embedding generation failed, using keyword matching" |
| Learnings corrupt | "Learnings file corrupted. Creating backup and reinitializing." |
