---
description: "Resume from last checkpoint"
argument-hint: ""
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash, Task]
---

# /orc:resume Command

## Usage

```
/orc:resume
```

## Description

Resumes execution from the last checkpoint. Identifies incomplete stories and continues from there.

## Behavior

1. Load latest checkpoint
2. Validate checkpoint against current codebase
3. Identify incomplete stories
4. Resume execution from first incomplete story
5. Continue normal execution flow

## Output

```
🔄 Resuming execution...

Checkpoint loaded: E2-F1-S3-stop.json
  ├─ Saved: 2025-01-30T14:30:00Z
  ├─ Phase: execute
  ├─ Position: E2-F1-S3 (completed)
  └─ Validating codebase state...

✓ Codebase matches checkpoint state

Resuming from E2-F1-S4...

[E2-F1-S4] Creating user update endpoint
  ├─ REASON: Loading previous context...
  ...
  └─ COMPLETE ✓ [6.1s]

... (continues execution) ...
```

## Checkpoint Validation

Before resuming, validates:
1. All checkpoint files still exist
2. File hashes match (no external modifications)
3. Dependencies still satisfied
4. No conflicting changes

If validation fails:

```
⚠️  Checkpoint validation failed

Issues detected:
  - src/models/User.ts modified externally
  - New file detected: src/routes/manual-fix.ts

Options:
  [C]ontinue anyway (may cause conflicts)
  [R]e-plan affected stories
  [A]bort and review manually

Choice:
```

## State Updates

- Restores state from checkpoint
- Updates `last_activity`
- Continues execution updates as normal

## Use Cases

- After `/orc:stop`
- After system interruption
- After manual review/modification
- After fixing blocked stories externally

## Error Handling

| Error | Response |
|-------|----------|
| No checkpoint | "No checkpoint found. Run /orc:run to start execution." |
| Corrupt checkpoint | "Checkpoint file corrupted. Manual recovery needed." |
| Validation failed | Shows validation issues with options |
| Already executing | "Execution already in progress. Use /orc:stop first." |
