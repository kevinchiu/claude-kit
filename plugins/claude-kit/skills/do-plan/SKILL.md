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
| File | Step B modifies file Step A creates |
| Code | Step B imports code Step A writes |
| Data | Step B needs Step A output |
| Order | Step B tests what Step A implements |

Independent if: different files AND no shared data AND no validation relationship.

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

### Subagent Prompt

```
Implement step [N]: [description]

Context:
- Structure: [from explore]
- Pattern: [path/to/similar/code.ts]
- Related files: [from explore]

Files: [list]
Requirements: [from plan]

Implement and report what you did.
```

## Phase 4: Complete

1. Update progress to N/N
2. Add completion timestamp
3. Move to `.claude/plans/archive/[name].state.md`

## Handling Blocks

When subagent reports blocked:
1. Mark step blocked in state
2. Record reason
3. Continue independent steps
4. After group completes, assess:
   - Resolvable: fix and retry
   - Needs user: ask (only exception to zero-prompt)

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
