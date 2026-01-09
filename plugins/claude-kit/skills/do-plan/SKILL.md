---
name: do-plan
description: Autonomous parallel plan execution with dependency-aware critical path optimization. Trigger: "do the plan", "execute", "implement", "proceed", "let's do it", "continue plan", "resume".
---

# Do-Plan Skill

Autonomous parallel execution. Only main agent updates state file. Subagents do work via Task tool.

## Execution Phases

1. EXPLORE - Read from plan or launch agents (if missing)
2. SETUP - Create state file, analyze dependencies, build execution groups
3. EXECUTE - Launch parallel subagents per group
4. COMPLETE - Update state, archive
5. REVIEW - Multi-aspect code review (if available)

## Phase 1: Explore

**If plan has "Exploration Context" section:** Read it, skip exploration.

**If missing:** Invoke the parallel-explore skill to launch 6 agents and gather codebase context.

## Phase 2: Setup

### 2.1 Create State File

Write to `.claude/plans/[plan-name].state.md`:

```
# Plan: [Name]
Created: [date]
Updated: [datetime]

## Progress: 0/N

## Execution Strategy

Dependencies:
- Step 1: None
- Step 2: None
- Step 3: Step 1
- Step 4: Steps 1,2,3

Groups:
- Group 1 (parallel): Steps 1, 2
- Group 2 (sequential): Step 3
- Group 3 (after all): Step 4

## Steps

### Step 1: [Description]
Status: pending
Dependencies: None
Files: [paths]
Agent: general-purpose (opus)

### Step 2: [Description]
Status: pending
Dependencies: None
Files: [paths]
Agent: general-purpose (opus)
```

### 2.2 Analyze Dependencies

Dependency types:

| Type | Detection |
|------|-----------|
| File overlap | Steps modify same file(s) → sequential |
| File | Step B modifies file Step A creates |
| Code | Step B imports code Step A writes |
| Data | Step B needs Step A output |
| Order | Step B tests what Step A implements |

**File overlap rule:** If two steps touch ANY of the same files, they MUST be sequential (not parallel). This prevents merge conflicts and inconsistent changes.

Independent if: no overlapping files AND no shared data AND no validation relationship.

### 2.3 Build Execution Groups

```
Group 1: [independent steps] - launch parallel
Group 2: [depends on Group 1] - launch after Group 1
Group 3: [tests/builds] - run in background
```

## Phase 3: Execute

For each group:

1. Mark all steps in group: `Status: in_progress`
2. Save state file
3. Launch ALL subagents in ONE message (parallel)
4. Wait for results
5. For each result:
   - Success: `Status: completed`, add notes
   - Blocked: `Status: blocked`, add reason
6. Save state file
7. Next group (or handle blocks)

### Agent Selection

| Step Type | Agent | Model |
|-----------|-------|-------|
| Implement feature | general-purpose | opus |
| Fix bug | general-purpose | opus |
| Write tests | general-purpose | opus |
| Explore/research | Explore | sonnet |
| Simple changes | general-purpose | haiku |
| Complex architecture | general-purpose | opus |
| Run tests | general-purpose | haiku |

### Pre-flight (per step)

Before launching each step's agent, verify:
1. All listed files exist (or are being created)
2. Parent directories exist for new files

If files are missing and not being created, mark step blocked.

### Subagent Prompt

```
Implement step [N]: [description]

Context:
- Structure: [from explore]
- Pattern: [path/to/similar/code.ts]
- Related files: [from explore]

Files: [list]
Requirements: [from plan]

Parallel work (FYI): [other steps in this group and their files]

Guidelines:
- If modifying function/method signatures, check for callers first
- Avoid modifying files assigned to parallel agents unless necessary
- Report any files you modified beyond your assigned list

Implement and report what you did.
```

## Phase 4: Complete

1. Update progress to N/N
2. Add completion timestamp
3. Move to `.claude/plans/archive/[name].state.md`

## Phase 5: Review

### Static Analysis

Run any available static analysis tools on changed files:
- Type checkers: `tsc --noEmit`, `mypy`, `pyright`
- Linters: `eslint`, `ruff`, `clippy`
- Build: `npm run build`, `cargo check`

Fix issues automatically. If unfixable, report to user.

### Code Review (if available)

If a multi-aspect code review agent/skill is available (e.g., `pr-review-toolkit:review-pr`, `feature-dev:code-reviewer`), invoke it on all changed files.

Review should cover:
- Code quality and style
- Silent failures and error handling
- Test coverage gaps
- Type design issues

Report findings to user. Fix critical issues automatically if straightforward.

## Handling Failures

### Blocked Steps

When subagent reports blocked:
1. Mark step blocked in state
2. Record reason
3. Continue independent steps
4. After group completes, assess:
   - Resolvable: fix and retry
   - Needs user: ask (only exception to zero-prompt)

### Context Exhaustion

When subagent fails due to context limit:

1. Check for partial progress (git diff, file changes)
2. Analyze the step and split into smaller substeps (N.1, N.2, ...)
3. Update state file with new substeps
4. Retry with first substep
5. If substep also exhausts: split again recursively

Main agent decides how to split based on the specific task.

## Resuming

On "continue plan":
1. Read state file
2. Find steps marked in_progress or pending
3. Rebuild groups from remaining steps
4. Continue from Phase 3

## Core Rules

1. Only main agent writes state file
2. If steps CAN parallel, they MUST parallel
3. Never prompt during execution (except blocks)
4. Save state after every status change
