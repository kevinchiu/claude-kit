---
description: Plan and execute a task
argument-hint: [task description]
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Task, TodoWrite, AskUserQuestion
---

Plan: $ARGUMENTS

Use the plan skill to create an implementation plan with explicit user approval.

**Do NOT use EnterPlanMode or ExitPlanMode** - this skill uses AskUserQuestion for approval gates.
