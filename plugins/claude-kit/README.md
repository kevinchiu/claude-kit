# claude-kit

Autonomous parallel plan execution with dependency-aware critical path optimization.

## What It Does

**Plan** → **Execute** workflow:
1. **Plan**: Parallel exploration, architecture design, user approval
2. **Execute**: Autonomous parallel execution with state tracking

## Skills

### parallel-explore

Parallel codebase exploration with 6 specialized agents. Used by make-plan and do-plan.

**Agents:** structure, patterns, dependencies, types, tests, config

### make-plan

Creates implementation plans with parallel exploration and user approval.

**How it works:**
1. Launches parallel Explore agents to understand codebase
2. Designs implementation approach with Plan agents
3. Writes plan to `.claude/plans/[name].md`
4. Asks for user approval

**Usage:**
```
/make-plan [task description]
```

### do-plan

Executes plans in parallel with state tracking.

**How it works:**
1. Reads exploration context from plan (or explores if missing)
2. Creates state file at `.claude/plans/[name].state.md`
3. Analyzes dependencies and builds execution groups
4. Launches parallel Task agents for independent steps
5. Tracks progress with status markers (pending, in_progress, completed, blocked)
6. Resumes from last checkpoint if interrupted

**Usage:**
```
/do-plan [plan-file]
```
Or say: "do the plan", "do-plan", "proceed", "continue plan"

## Commands

| Command | Description |
|---------|-------------|
| `/make-plan [task]` | Create a plan for a task |
| `/do-plan [file]` | Execute a plan file |

## Hooks

**SessionStart**: Checks for incomplete plans and notifies you to resume.
