# claude-kit

Autonomous parallel plan execution with dependency-aware critical path optimization.

## What It Does

**Plan** → **Execute** workflow:
1. **Plan**: Parallel exploration, architecture design, user approval
2. **Execute**: Autonomous parallel execution with state tracking

## Skills

### plan

Creates implementation plans with parallel exploration and user approval.

**How it works:**
1. Launches parallel Explore agents to understand codebase
2. Designs implementation approach with Plan agents
3. Writes plan to `.claude/plans/[name].md`
4. Exits plan mode for user approval

**Usage:**
```
/plan [task description]
```

### do-plan

Executes plans in parallel with state tracking.

**How it works:**
1. Launches parallel Explore agents to understand codebase
2. Creates state file at `.claude/plans/[name].state.md`
3. Analyzes dependencies and builds execution groups
4. Launches multiple Task agents for independent steps
5. Tracks progress with status markers (⏳ pending, 🔄 in_progress, ✅ completed, ❌ blocked)
6. Resumes from last checkpoint if interrupted

**Usage:**
```
/do-plan [plan-file]
```
Or say: "do the plan", "execute", "proceed", "continue plan"

## Commands

| Command | Description |
|---------|-------------|
| `/plan [task]` | Create a plan for a task |
| `/do-plan [file]` | Execute a plan file |
