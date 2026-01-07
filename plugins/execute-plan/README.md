# execute-plan

Claude Code plugin for executing implementation plans with session persistence.

## Features

- State file tracking in `.claude/plans/*.state.md`
- Session crash recovery
- Resumable execution across sessions
- Progress checkpointing after each step
- **SessionStart hook** - automatically detects in-progress plans on startup

## Installation

```bash
# Test locally
claude --plugin-dir /path/to/execute-plan-plugin

# Or install from marketplace (once published)
/plugin install execute-plan@your-marketplace
```

## Usage

### Slash Command

```
/execute-plan <plan-file>
```

If no plan file specified, checks `.claude/plans/*.state.md` for in-progress plans.

### Natural Language

The skill activates when you say:
- "execute the plan"
- "implement the plan"
- "proceed with the plan"
- "continue plan"
- "resume plan"

### Plan Commands

While executing:
- "Continue the plan" - Resume from current step
- "Show plan status" - Display progress summary
- "Skip step N" - Mark step as skipped, move on
- "Restart step N" - Reset step to pending, re-execute
- "Abort plan" - Stop execution (state preserved)
- "Complete plan" - Mark current plan as done, run cleanup

## State File Format

Plans are tracked in `.claude/plans/[plan-name].state.md`:

```markdown
# Plan: [Name]
Created: [date]
Last Updated: [date time]

## Progress: 0/N steps completed

## Steps

### Step 1: [Description]
**Status:** pending | in_progress | completed | blocked
**Files:** [files to modify]
```

## License

MIT
