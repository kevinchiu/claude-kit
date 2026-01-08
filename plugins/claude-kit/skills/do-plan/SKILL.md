---
name: do-plan
description: Autonomous parallel plan execution with dependency-aware critical path optimization. Analyzes step dependencies, parallelizes independent work using subagents, and tracks state for crash recovery. Trigger phrases: "do the plan", "execute", "implement", "proceed", "let's do it", "continue plan", "resume".
---

# Do Plan

Autonomous parallel plan execution with dependency-aware critical path optimization.

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                   MAIN AGENT                        │
│  - Owns state file (only writer)                    │
│  - Analyzes dependencies                            │
│  - Launches subagents via Task tool                 │
│  - Collects results and updates state              │
└─────────────────────────────────────────────────────┘
          │               │               │
          ▼               ▼               ▼
    ┌──────────┐   ┌──────────┐   ┌──────────┐
    │ Task     │   │ Task     │   │ Task     │
    │ Agent 1  │   │ Agent 2  │   │ Agent 3  │
    │ (Step A) │   │ (Step B) │   │ (Step C) │
    └──────────┘   └──────────┘   └──────────┘
    Independent steps run in parallel
```

**Key principle:** Subagents do work and return results. Only the main agent updates the state file.

---

## MANDATORY FIRST STEPS

**Complete these before ANY implementation:**

### Step 1: Explore the Codebase (parallel)

Launch multiple `Explore` agents in parallel to understand the codebase:

```
Single message with multiple Task tool calls:

Task 1 (structure):
- subagent_type: "Explore"
- prompt: "Map the directory structure and file organization.
  Identify: main source dirs, config files, test locations, build outputs.
  Report: tree structure, key directories, entry points."

Task 2 (patterns):
- subagent_type: "Explore"
- prompt: "Find existing patterns for [relevant area from plan].
  Search for similar implementations to: [list what plan needs to build].
  Report: pattern locations, code examples, conventions used."

Task 3 (dependencies):
- subagent_type: "Explore"
- prompt: "Analyze dependencies between these files: [list files from plan].
  Identify: imports, shared types, call relationships.
  Report: dependency graph, execution order constraints."
```

Use exploration results to inform dependency analysis and provide context to subagents.

### Step 2: Create State File

Create `.claude/plans/[plan-name].state.md` using the template below. This must exist before implementation begins.

### Step 3: Analyze Dependencies (using exploration results)

For each step, determine its dependencies using these criteria:

| Dependency Type | How to Detect |
|-----------------|---------------|
| **File dependency** | Step B modifies a file that Step A creates |
| **Code dependency** | Step B imports/uses code that Step A writes |
| **Data dependency** | Step B needs output/result from Step A |
| **Order dependency** | Step B tests/validates what Step A implements |

**Steps are independent if:** They touch different files AND don't share data AND neither validates the other.

### Step 4: Build Execution Groups

Organize steps into groups:

```
Group 1 (parallel):  [independent steps] → launch simultaneously
Group 2 (parallel):  [steps dependent on Group 1] → launch after Group 1 completes
Group 3 (background): [tests/builds] → run while other work continues
```

### Step 5: Launch Parallel Subagents

For each group of independent steps, launch ALL subagents in a SINGLE message using multiple Task tool calls.

---

## Execution Flow

### Phase 1: Exploration
1. Read the plan file
2. Launch parallel Explore agents (structure, patterns, dependencies)
3. Collect exploration results

### Phase 2: Setup
1. Create state file with all steps marked ⏳ pending
2. Use exploration results to analyze dependencies
3. Document execution groups in state file
4. Save state file

### Phase 3: Execute Groups
For each execution group:

1. **Mark steps in_progress** - Update state file for all steps in this group
2. **Launch subagents in parallel** - Single message with multiple Task calls
3. **Wait for results** - All subagents in group must complete
4. **Process results** - For each completed subagent:
   - If success: Mark step ✅ completed, record notes
   - If blocked: Mark step ❌ blocked, record reason
5. **Save state file**
6. **Proceed to next group** (or handle blocks)

### Phase 4: Completion
1. Update progress to N/N
2. Generate summary
3. Archive state file

---

## Launching Subagents

Use the Task tool with appropriate agent types:

### Agent Type Selection

| Step Type | Agent Type | Model |
|-----------|------------|-------|
| Implement feature | `general-purpose` | sonnet |
| Fix bug | `general-purpose` | sonnet |
| Explore/research | `Explore` | sonnet |
| Simple file changes | `general-purpose` | haiku |
| Complex architecture | `general-purpose` | opus |
| Run tests | `Bash` | haiku |

### Subagent Prompt Template

```
Implement step [N] of the plan: [step description]

Context from exploration:
- Codebase structure: [summary from structure Explore agent]
- Pattern to follow: [path/to/example.ts from patterns Explore agent]
- Related files: [from dependencies Explore agent]

Files to modify: [list]

Requirements:
- [specific requirements from plan]

Do NOT create a state file. Just implement the step and report what you did.
```

### Parallel Launch Example

To run steps 1, 2, 3 in parallel, send ONE message with THREE Task tool calls:

```
<Task tool call 1: Step 1 prompt, subagent_type="general-purpose", model="sonnet">
<Task tool call 2: Step 2 prompt, subagent_type="general-purpose", model="sonnet">
<Task tool call 3: Step 3 prompt, subagent_type="Explore", model="haiku">
```

---

## State File Template

Write to `.claude/plans/[plan-name].state.md`:

```markdown
# Plan: [Name]
Created: [date]
Last Updated: [date time]

## Progress: 0/N steps completed

## Execution Strategy

**Dependency analysis:**
- Step 1: No dependencies (independent)
- Step 2: No dependencies (independent)
- Step 3: Depends on Step 1 (uses code from Step 1)
- Step 4: Depends on Steps 1, 2, 3 (integration test)

**Execution groups:**
- Group 1 (parallel): Steps 1, 2
- Group 2 (sequential): Step 3 (after Group 1)
- Group 3 (after all): Step 4 (tests)

## Steps

### Step 1: [Description]
**Status:** ⏳ pending
**Dependencies:** None
**Files:** [files to modify]
**Agent:** general-purpose (sonnet)

### Step 2: [Description]
**Status:** ⏳ pending
**Dependencies:** None
**Files:** [files to modify]
**Agent:** general-purpose (sonnet)

### Step 3: [Description]
**Status:** ⏳ pending
**Dependencies:** Step 1
**Files:** [files to modify]
**Agent:** general-purpose (sonnet)
```

---

## Tool Priority

Prefer existing tools over bash commands:

| Task | Use This | Not This |
|------|----------|----------|
| Read file | `Read` tool | `cat`, `head`, `tail` |
| Search content | `Grep` tool | `grep`, `rg` |
| Find files | `Glob` tool | `find`, `ls` |
| Edit file | `Edit` tool | `sed`, `awk` |
| Write file | `Write` tool | `echo >`, heredoc |
| Run commands | `Bash` tool | Only when necessary |

---

## Handling Blocks

When a subagent reports it's blocked:

1. Mark step ❌ blocked in state file
2. Record the block reason
3. **Continue with independent steps** - Don't stop everything
4. After current group completes, assess blocks:
   - Can you resolve it? → Fix and retry
   - Need user input? → Ask user (only exception to zero-prompt)

---

## Resuming a Plan

**On "continue plan":**

1. Read state file
2. Find resume point:
   - Steps marked 🔄 in_progress → Reassess, may need restart
   - Steps marked ⏳ pending → Ready to execute
3. Rebuild execution groups from remaining steps
4. Continue execution flow from Phase 2

---

## Core Principles

1. **State file ownership** - Only main agent writes to state file
2. **Parallel by default** - If steps CAN run in parallel, they MUST
3. **Zero-prompt execution** - Never ask user during execution (except blocks)
4. **Immediate state saves** - Save after every status change
5. **Tool preference** - Use Read/Edit/Glob/Grep over bash equivalents

---

## Commands

The user can say:
- "Continue the plan" - Resume from current state
- "Show plan status" - Display progress summary
- "Skip step N" - Mark as skipped, continue
- "Restart step N" - Reset to pending, re-execute
- "Abort plan" - Stop execution (state preserved)
