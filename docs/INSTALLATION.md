# Installing ORC from Claude Code Marketplace

Install ORC in under a minute from the Claude Code marketplace.

## Prerequisites

You need [Claude Code](https://docs.anthropic.com/claude-code) installed and authenticated.

```bash
# Verify Claude Code is installed
claude --version
```

If not installed, see [Claude Code installation docs](https://docs.anthropic.com/claude-code/getting-started).

## Install from Marketplace

### Step 1: Add the Plugin

```bash
claude plugins add twofoldtech-dakota/ORC
```

### Step 2: Verify Installation

```bash
claude plugins list
```

You should see:
```
orc (v1.0.0) - Multi-Agent Orchestration System
```

### Step 3: Test It Works

```bash
claude
```

Then in the Claude Code session:
```
> /orc status
```

Expected output:
```
📊 ORC Status
No active session.
Run /orc plan <goal> to start.
```

## Start Using ORC

Navigate to any project and create your first plan:

```bash
cd your-project
claude
```

```
> /orc plan "Add user authentication with JWT"
> /orc show
> /orc approve
> /orc run
```

ORC will analyze your codebase, create a structured plan, and execute it autonomously.

## Updating

```bash
claude plugins update orc
```

## Uninstalling

```bash
claude plugins remove orc
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Plugin not found | Run `claude plugins add twofoldtech-dakota/ORC` again |
| Commands not recognized | Restart your Claude Code session |
| Version mismatch | Run `claude plugins update orc` |

## Next Steps

- [Quick Start](../README.md#-quick-start) - Complete walkthrough
- [Commands](COMMANDS.md) - All `/orc` commands
- [Architecture](ARCHITECTURE.md) - How ORC works
