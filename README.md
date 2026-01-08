# claude-kit

Autonomous parallel plan execution with dependency-aware critical path optimization.

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

## What It Does

**Plan** → **Execute** workflow:
1. **make-plan**: Parallel exploration, architecture design, user approval
2. **do-plan**: Autonomous parallel execution with state tracking

## Skills

| Skill | Description |
|-------|-------------|
| parallel-explore | 6 specialized agents for codebase exploration |
| make-plan | Create implementation plans with user approval |
| do-plan | Execute plans in parallel with state tracking |

## Commands

| Command | Description |
|---------|-------------|
| `/make-plan [task]` | Create a plan for a task |
| `/do-plan [file]` | Execute a plan file |

## Usage

```
/make-plan add user authentication
```

After approval:

```
/do-plan
```

Or say: "do the plan", "proceed", "continue plan"

## Hooks

**SessionStart**: Checks `.claude/plans/` for incomplete `.state.md` files and notifies you if there's a plan to resume. No auto-resume - waits for your go-ahead.
