---
name: do-plan
description: Persistent parallel plan execution. Use when user says "do the plan", "execute", "implement", "proceed", "let's do it", "continue plan", or "resume". Tracks progress in state files for crash recovery.
---

# Do Plan

Persistent plan execution with state tracking across sessions.

## MANDATORY FIRST STEPS (Do These Before ANY Implementation)

**You MUST complete steps 1-3 before writing any code:**

### Step 1: Create State File IMMEDIATELY

Before doing anything else, create `.claude/plans/[plan-name].state.md` using the template below. This is NON-NEGOTIABLE - the state file must exist before any implementation begins.

### Step 2: Critical Path Analysis

Analyze the plan steps and identify:
- **Independent steps** that can run in parallel (no dependencies between them)
- **Dependent steps** that must run sequentially (step B needs output from step A)
- **Long-running steps** (tests, builds) that should run in background

Document this analysis in the state file under "## Execution Strategy".

### Step 3: Launch Parallel Subagents

For independent steps, you MUST use Task tool to launch multiple subagents in a SINGLE message. Do not execute steps sequentially when they can be parallelized.

Example: If steps 1, 2, 3 are independent, launch 3 Task agents in one message block.

---

## Core Principles

1. **Every state change must be saved immediately** - sessions can end at any moment.

2. **Zero-prompt execution** - All blockers are resolved during planning. Execution never prompts.

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

## Pre-Execution Checks

Before creating state file, verify the plan is executable:

1. **Plan completeness** - Each step has:
   - Target file path
   - Pattern/example to follow (or reference in CLAUDE.md)
   - Data sources / dependencies

2. **Credentials/secrets** - For each external service:
   - Use Grep to check config files have required keys
   - Use Bash to verify env vars exist: `test -n "$VAR_NAME"`
   - Optionally test connectivity

3. **Codebase patterns** - Read 1-2 similar implementations per step type

If anything missing → reject plan with specific requirements, don't start execution.

### State File Template

Write this to `.claude/plans/[plan-name].state.md`:

```markdown
# Plan: [Name]
Created: [date]
Last Updated: [date time]

## Progress: 0/N steps completed

## Execution Strategy

**Parallel groups:**
- Group 1 (parallel): Steps 1, 2, 3 - no dependencies
- Group 2 (sequential): Step 4 depends on 1, 2
- Group 3 (background): Step 5 (tests) - run while doing Group 2

**Model assignment:**
- Steps 1, 2: sonnet (mechanical changes)
- Step 3: haiku (simple lookup)
- Step 4: opus (complex reasoning)

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
- Research using Glob, Grep, Read to understand patterns
- Document non-obvious decisions in state file notes

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
4. Ask user (independent steps already running in parallel)

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

## Prompt Avoidance

**During execution:** Research → Decide → Document → Proceed

Before each step, read 2-3 similar implementations. This answers most questions.

**When uncertain, use defaults:**
- Naming: match nearest similar code
- Errors: match existing patterns
- Tests: unit tests for logic only
- Style: simpler over abstract

**Assume and document** (`Assumed: X because Y` in state file) unless:
- Data loss risk
- Security impact
- Public API change

**Prompt only** when blocked and cannot proceed.

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
