---
description: Create a plan for a task
argument-hint: [task description]
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Task, TodoWrite, AskUserQuestion
---

Make plan: $ARGUMENTS

Use the make-plan skill to create an implementation plan with explicit user approval.

**Do NOT use EnterPlanMode or ExitPlanMode** - this skill uses AskUserQuestion for approval gates.
