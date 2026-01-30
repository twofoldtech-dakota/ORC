---
description: "Stop execution gracefully"
argument-hint: ""
allowed-tools: [Read, Write]
---

# /orc stop Command

## Usage

```
/orc stop
```

## Description

Gracefully stops execution, saving current state and checkpoint. Does not abort mid-story—waits for current story to complete or fail.

## Behavior

1. Signal stop to Orchestrator
2. Wait for current story to complete/fail
3. Save checkpoint
4. Update state to paused
5. Show summary

## Output

```
⏸  Stopping execution...

Waiting for current story to complete...

[E2-F1-S3] Creating user endpoint
  ├─ ... (continues until done)
  └─ COMPLETE ✓ [5.2s]

✓ Execution stopped gracefully

Progress saved:
  ├─ Epics: 1/2 completed
  ├─ Features: 4/7 completed (E2-F1 in progress)
  ├─ Stories: 13/23 completed
  ├─ Checkpoint: E2-F1-S3-stop.json
  └─ Phase: execute (paused)

Run /orc resume to continue from here
Run /orc status to see current state
```

## State Updates

- Creates checkpoint at current position
- Does not change `phase` (remains `execute`)
- Records stop time in state

## Use Cases

- Need to pause for external reason
- Want to review progress mid-execution
- Resource constraints require stopping
- User wants to make manual changes

## Notes

- If no execution in progress, shows message: "No execution in progress."
- Does not lose any work—all completed stories preserved
- Resume continues exactly where stopped

## Error Handling

| Error | Response |
|-------|----------|
| Not executing | "No execution in progress." |
| Story stuck | "Current story not responding. Force stop? [y/N]" |
