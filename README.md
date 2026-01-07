# claude-kit

Powerful plugins for Claude Code.

- ~~beads~~
- ~~agent mailboxes~~
- ~~dev containers~~
- ~~worktrees~~
- durable parallel execution across sessions

## Installation

1. Run `/plugin` in Claude Code
2. Select **Manage Marketplaces** → **Add Marketplace**
3. Enter `kevinchiu/claude-kit`
4. Browse and install plugins from the marketplace

## Migration from do-plan

If you previously installed `do-plan`, uninstall it and install `claude-kit`:

```
/plugin → Uninstall → do-plan
/plugin → Install → claude-kit
```

## Plugins

| Plugin | Description |
|--------|-------------|
| [claude-kit](./plugins/claude-kit) | Claude Code productivity toolkit |

## claude-kit

### do-plan

Durable plan execution with state tracking in `.claude/plans/*.state.md`, crash recovery, and auto-resume on session start.

```
/do-plan <plan-file>
```

Or say: "do the plan", "execute", "proceed", "continue plan"

### Hooks

**SessionStart**: On every new session, Claude checks `.claude/plans/` for incomplete `.state.md` files and notifies you if there's a plan to resume. No auto-resume - waits for your go-ahead.
