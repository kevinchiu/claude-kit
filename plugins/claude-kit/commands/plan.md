---
description: Plan and execute a task
argument-hint: [task description]
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Task, TodoWrite, AskUserQuestion, EnterPlanMode, ExitPlanMode
---

Plan: $ARGUMENTS

## Instructions

1. **Enter plan mode** with EnterPlanMode

2. **Phase 1: Explore** - Launch up to 3 Explore agents in parallel:
   - Agent 1: Map directory structure and file organization
   - Agent 2: Find existing patterns for the relevant area
   - Agent 3: Analyze dependencies between files to modify

3. **Phase 2: Design** - Launch Plan agent(s) to design implementation:
   - Provide exploration results as context
   - Request detailed implementation plan

4. **Phase 3: Review** - Ensure alignment:
   - Read critical files identified by agents
   - Use AskUserQuestion to clarify remaining questions

5. **Phase 4: Write plan** to `.claude/plans/[plan-name].md`:
   - Each step needs: target file, pattern/example, dependencies
   - Include verification section (how to test)

6. **Exit plan mode** with ExitPlanMode for user approval

7. **After approval** - Invoke do-plan skill to execute
