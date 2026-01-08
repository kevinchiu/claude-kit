# claude-kit

Autonomous parallel plan execution with dependency-aware critical path optimization.

## What It Does

Takes an approved plan and executes it autonomously:
- **Analyzes dependencies** between steps
- **Parallelizes independent steps** using subagents
- **Optimizes for total time** (critical path), not sequential execution
- **Tracks state** for crash recovery and session resumption

## Skills

### do-plan

Executes plans in parallel with state tracking.

**How it works:**
1. Creates state file at `.claude/plans/[name].state.md`
2. Performs critical path analysis (which steps can run in parallel)
3. Launches multiple Task agents for independent steps
4. Tracks progress with status markers (⏳ pending, 🔄 in_progress, ✅ completed, ❌ blocked)
5. Resumes from last checkpoint if interrupted

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
