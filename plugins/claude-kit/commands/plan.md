---
description: Plan and execute a task
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Task, TodoWrite, AskUserQuestion, EnterPlanMode, ExitPlanMode
---

Plan: $ARGUMENTS

## Instructions

1. Use EnterPlanMode to enter planning mode

2. While planning, ensure the plan includes for each step:
   - Target file path
   - Pattern/example to follow (or note in CLAUDE.md)
   - Data sources / dependencies

3. Run pre-flight checks before finalizing:
   - Use Grep to verify config files have required keys
   - Use Bash to verify env vars exist for external services
   - Read 1-2 similar implementations per step type

4. Write plan to `.claude/plans/[plan-name].md`

5. Use ExitPlanMode for user approval

6. After approval, invoke the do-plan skill to execute
