---
name: do-plan
description: Persistent plan execution. Use when user says "do the plan", "execute", "implement", "proceed", "let's do it", "continue plan", or "resume". Tracks progress in state files for crash recovery.
---

# Do Plan

Persistent plan execution with state tracking across sessions.

## Core Principles

1. **Every state change must be saved immediately** - sessions can end at any moment.

2. **Autonomous execution over prompting** - Make informed decisions independently:
   - Research and explore the codebase to understand context
   - Use existing patterns and conventions found in the code
   - Make reasonable implementation choices without asking
   - Only prompt user when genuinely blocked or facing critical architectural decisions
   - Document decisions made in state file notes for transparency

## When to Activate

**DO activate when:**
- User explicitly asks to execute a plan
- User says "execute", "implement", "proceed" after planning
- User runs `/execute-plan` command
- User says "continue plan" and state file exists

**DO NOT activate when:**
- No plan exists (don't create one automatically)
- User is just discussing or brainstorming
- Plan mode is still active (planning not finished)
- User hasn't approved the plan

**If no plan exists:** Simply inform the user "No plan found to execute. Please create a plan first or describe what you want to implement."

## Starting a New Plan

When given an approved plan to execute:

1. **Parse into steps** - Break into atomic, completable units
2. **Create state file** at `.claude/plans/[plan-name].state.md`
3. **Initialize all steps** as pending
4. **Begin execution** from step 1

### State File Template

Write this to `.claude/plans/[plan-name].state.md`:

```markdown
# Plan: [Name]
Created: [date]
Last Updated: [date time]

## Progress: 0/N steps completed

## Steps

### Step 1: [Description]
**Status:** ⏳ pending
**Blocked by:** None
**Files:** [files to modify]

[Repeat for all steps]
```

## Executing Steps

For each step:

### Before Starting
1. Update status to `🔄 in_progress`
2. Add `Started: [timestamp]`
3. **Save state file immediately**

### During Execution
- Implement the step autonomously
- If complex, add progress notes to state file periodically
- **Research before asking**: Use Glob, Grep, Read to understand patterns
- **Decide independently**: Choose implementation approaches based on codebase conventions
- **Document decisions**: Record non-obvious choices in state file notes

### After Completing
1. Update status to `✅ completed`
2. Add `Completed: [timestamp]`
3. Add `Notes:` with summary of what was done
4. Update progress counter
5. **Save state file immediately**

### On Failure/Block
1. Update status to `❌ blocked`
2. Add `Blocked reason:` explanation
3. **Save state file immediately**
4. Inform user, ask how to proceed

## Resuming a Plan (Hybrid Approach)

**On session start** (via CLAUDE.md instruction):
1. Check `.claude/plans/` for any `.state.md` files
2. If incomplete plans exist, **notify** (don't auto-resume):
   > "📋 Found in-progress plan: '[name]' (5/12 steps). Say 'continue plan' to resume."
3. Wait for user confirmation before resuming

**When user says "continue plan":**
1. **Read state file**
2. **Summarize progress**:
   - "Resuming plan [name]: 5/12 steps completed"
   - "Last completed: Step 5 - Added user authentication"
   - "Next step: Step 6 - Create login endpoint"
3. **Find resume point**:
   - If step is `in_progress`: Assess partial work, complete or restart
   - If step is `pending`: Start fresh
4. **Continue execution**

## Execution Optimization

**Critical path optimization** - minimize total execution time:

1. **Parallelize independent steps** - Use Task agents for steps that don't depend on each other
   - Launch multiple agents in a single message for parallel execution
   - Each agent works on a separate step simultaneously

2. **Model selection by complexity:**
   - `haiku`: Quick lookups, simple file reads, status checks
   - `sonnet`: Exploration, mechanical changes, most implementation work
   - `opus`: Complex reasoning, architectural decisions, tricky bugs

3. **Background long-running tasks:**
   - Start tests/builds in background
   - Continue with independent work while waiting
   - Check results when needed

4. **Batch file operations:**
   - Read multiple files in parallel when exploring
   - Make independent edits in parallel

## Autonomous Decision-Making

**DO decide independently:**
- Implementation details (variable names, code structure)
- Which existing patterns/utilities to use
- Test approach and coverage
- Error handling strategy based on codebase norms
- File organization following project conventions

**DO research first:**
- Read similar existing code before implementing
- Check CLAUDE.md and project docs for conventions
- Look at existing tests to understand testing patterns
- Explore imports and dependencies before adding new ones

**DO NOT prompt unless:**
- Genuinely blocked (missing credentials, unclear requirements)
- Major architectural decision not covered by the plan
- Discovered significant scope change that affects other parts
- Found security concern or breaking change

**DO document in state file:**
- Decisions made and rationale
- Patterns discovered and followed
- Any assumptions made

## Step Granularity Guidelines

Good steps are:
- **Atomic** - Can be completed in one focused effort
- **Verifiable** - Clear success criteria
- **Self-contained** - Minimal dependencies on other steps
- **Resumable** - If interrupted, can restart cleanly

Split if step would take >15 minutes or touch >3 files.

## Plan Completion & Cleanup

When all steps are completed:

1. **Update final status** in state file:
   - Change progress to "N/N steps completed"
   - Add completion timestamp
   - Mark plan status as `✅ COMPLETED`

2. **Generate completion summary:**
   - List all completed steps with notes
   - Total duration (created → completed)
   - Key decisions made
   - Any issues encountered and resolutions

3. **Archive state file:**
   - Move from `.claude/plans/[name].state.md` to `.claude/plans/archive/[name].state.md`
   - Keep for reference but don't trigger resume notifications

4. **Cleanup:**
   - Remove any temporary files created during execution
   - Clear related TodoWrite items

## Commands

The user can say:
- "Continue the plan" - Resume from current step
- "Show plan status" - Display progress summary
- "Skip step N" - Mark step as skipped, move on
- "Restart step N" - Reset step to pending, re-execute
- "Abort plan" - Stop execution (state preserved)
- "Complete plan" - Mark current plan as done, run cleanup
