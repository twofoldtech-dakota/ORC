---
description: "Clear plan and state"
argument-hint: "[--all]"
allowed-tools: [Read, Write, Bash]
---

# /orc:clear Command

## Usage

```
/orc:clear
```

## Description

Clears the current plan and state. Requires confirmation to prevent accidental data loss. Does NOT delete learnings—patterns are preserved.

## Behavior

1. Prompt for confirmation
2. If confirmed:
   - Delete `.orc/plan/state.json`
   - Delete `.orc/plan/plan.json`
   - Delete `.orc/plan/epics/` contents
   - Delete `.orc/plan/deviations.json`
   - Delete `.orc/plan/scratchpad.json`
   - Delete `.orc/checkpoints/` contents
3. Preserve learnings and embeddings

## Confirmation Dialog

```
⚠️  Clear ORC State

This will delete:
  - Current plan (2 epics, 21 stories)
  - All checkpoints
  - Current session state
  - Deviation log
  - Scratchpad

This will NOT delete:
  - Learned patterns (23 patterns)
  - Embeddings
  - Generated code (already in your project)

Are you sure? [y/N]:
```

## Output (Confirmed)

```
✓ ORC state cleared

Deleted:
  - Plan: 2 epics, 7 features, 21 stories
  - Checkpoints: 5 files
  - Session state

Preserved:
  - Learnings: 23 patterns
  - Embeddings: 45 vectors

Run /orc:plan <goal> to start a new session.
```

## Output (Cancelled)

```
Clear cancelled. No changes made.
```

## Use Cases

- Starting fresh on a new project
- Abandoning current plan
- Resetting after failed session
- Cleaning up before sharing project

## Options

### Clear Everything

```
/orc:clear --all
```

```
⚠️  Clear ALL ORC Data

This will delete EVERYTHING including:
  - Current plan
  - All checkpoints
  - Learned patterns
  - Embeddings
  - All session history

This action cannot be undone.

Type 'DELETE ALL' to confirm:
```

## Error Handling

| Error | Response |
|-------|----------|
| No state exists | "Nothing to clear. No active ORC session." |
| In progress | "Cannot clear during execution. Run /orc:stop first." |
| Permission denied | "Cannot delete .orc directory. Check permissions." |
