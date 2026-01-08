---
name: plan
description: Plan implementation tasks with parallel exploration and explicit user approval. Use when user says "plan", "let's plan", "help me plan", or runs /plan command.
---

# Plan

Custom plan mode with parallel exploration and explicit user approval.

**Does NOT use EnterPlanMode/ExitPlanMode** - uses AskUserQuestion for explicit approval gates.

---

## Plan Workflow

```
┌─────────────────────────────────────────────────────┐
│  Phase 1: EXPLORE                                   │
│  Launch parallel Explore agents                     │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│  Phase 2: CLARIFY                                   │
│  AskUserQuestion to resolve ambiguities             │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│  Phase 3: DESIGN                                    │
│  Launch Plan agents to design implementation        │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│  Phase 4: WRITE PLAN                                │
│  Write plan to .claude/plans/[name].md              │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│  Phase 5: APPROVAL GATE                             │
│  AskUserQuestion: "Approve plan?"                   │
│  → Approved: Invoke do-plan skill                   │
│  → Rejected: Revise based on feedback               │
└─────────────────────────────────────────────────────┘
```

---

## Phase 1: Explore

**Goal:** Understand the codebase before designing.

Launch up to 3 Explore agents IN PARALLEL (single message, multiple Task calls):

```
Task 1 (structure):
- subagent_type: "Explore"
- model: "haiku"
- prompt: "Map the directory structure and file organization for this codebase.
  Identify: main source dirs, config files, test locations, build outputs.
  Report: tree structure, key directories, entry points."

Task 2 (patterns):
- subagent_type: "Explore"
- model: "sonnet"
- prompt: "Find existing patterns for [RELEVANT AREA FROM USER REQUEST].
  Search for similar implementations to: [WHAT USER WANTS TO BUILD].
  Report: pattern locations, code examples, conventions used."

Task 3 (dependencies):
- subagent_type: "Explore"
- model: "haiku"
- prompt: "Identify files likely to be modified for: [USER REQUEST].
  Analyze: imports, shared types, call relationships between them.
  Report: file list, dependency graph, modification order."
```

**When to use fewer agents:**
- 1 agent: Task is isolated to known files
- 2 agents: Moderate scope, skip structure if codebase is familiar
- 3 agents: Uncertain scope, unfamiliar codebase

---

## Phase 2: Clarify

**Goal:** Resolve ambiguities before designing.

Use **AskUserQuestion** to clarify:
- Unclear requirements
- Multiple valid approaches
- Scope boundaries
- Technical preferences

```
AskUserQuestion example:
- question: "Which authentication approach should we use?"
- options:
  - label: "JWT tokens (Recommended)"
    description: "Stateless, scalable, matches existing API patterns"
  - label: "Session cookies"
    description: "Simpler, but requires session storage"
```

**Skip this phase if:** Requirements are clear and unambiguous.

---

## Phase 3: Design

**Goal:** Create implementation approach.

Launch **Plan agent(s)** based on complexity:

| Complexity | Agents | Approach |
|------------|--------|----------|
| Simple | 1 | Single implementation plan |
| Medium | 2 | Two perspectives (e.g., minimal vs thorough) |
| Complex | 3 | Multiple approaches to compare |

```
Task (design):
- subagent_type: "Plan"
- model: "sonnet" (or "opus" for complex)
- prompt: "Design implementation for: [USER REQUEST]

  Context from exploration:
  - Structure: [summary from Task 1]
  - Patterns: [summary from Task 2]
  - Files: [summary from Task 3]

  User clarifications:
  - [answers from Phase 2]

  Provide:
  1. Step-by-step implementation plan
  2. Files to create/modify for each step
  3. Dependencies between steps
  4. Risks and mitigations"
```

---

## Phase 4: Write Plan

**Goal:** Write actionable plan file.

Write to `.claude/plans/[plan-name].md`:

```markdown
# Plan: [Name]

## Summary
[1-2 sentence overview of what this plan accomplishes]

## Context
- **Request:** [original user request]
- **Codebase patterns:** [key patterns identified]
- **Key files:** [main files to modify]

## Steps

### Step 1: [Description]
- **Files:** [exact paths]
- **Pattern:** [reference: path/to/similar/code.ts]
- **Dependencies:** None
- **Details:** [specific implementation notes]

### Step 2: [Description]
- **Files:** [exact paths]
- **Pattern:** [reference: path/to/similar/code.ts]
- **Dependencies:** Step 1
- **Details:** [specific implementation notes]

[Continue for all steps...]

## Verification
- [ ] [How to verify step 1]
- [ ] [How to verify step 2]
- [ ] [End-to-end test command]
```

**Plan quality checklist:**
- [ ] Each step has exact file paths
- [ ] Each step has pattern/example reference
- [ ] Dependencies between steps are explicit
- [ ] Verification steps are concrete and runnable

---

## Phase 5: Approval Gate

**Goal:** Get explicit user approval before execution.

Use **AskUserQuestion** for approval:

```
AskUserQuestion:
- question: "I've written the plan to .claude/plans/[name].md. Ready to execute?"
- options:
  - label: "Approve and execute"
    description: "Start autonomous parallel execution"
  - label: "Review first"
    description: "I'll read the plan file before deciding"
  - label: "Revise plan"
    description: "I have changes or feedback"
```

**Handle responses:**

| Response | Action |
|----------|--------|
| "Approve and execute" | Invoke do-plan skill immediately |
| "Review first" | Wait for user to read and respond |
| "Revise plan" | Ask for specific feedback, update plan, re-submit for approval |
| Other (custom input) | Treat as revision feedback |

---

## Tool Priority

Use existing tools, not bash equivalents:

| Task | Use | Don't Use |
|------|-----|-----------|
| Read file | `Read` | `cat`, `head`, `tail` |
| Search content | `Grep` | `grep`, `rg` |
| Find files | `Glob` | `find`, `ls` |
| Edit file | `Edit` | `sed`, `awk` |
| Write file | `Write` | `echo >`, heredoc |

---

## Invoking Execution

After approval, invoke do-plan skill by saying:

> "Do the plan" or "Execute the plan"

Or tell user to run:

```
/do-plan [plan-file]
```

---

## Example Flow

```
User: "Add user authentication to the API"

1. EXPLORE (parallel)
   → Structure agent: "src/ has routes/, models/, middleware/"
   → Patterns agent: "Found JWT auth in src/middleware/auth.ts"
   → Dependencies agent: "Need to modify: routes/user.ts, models/user.ts"

2. CLARIFY
   → AskUserQuestion: "JWT or session-based auth?"
   → User: "JWT"

3. DESIGN
   → Plan agent designs 5-step implementation

4. WRITE PLAN
   → Write to .claude/plans/user-auth.md

5. APPROVAL
   → AskUserQuestion: "Ready to execute?"
   → User: "Approve and execute"

6. EXECUTE
   → Invoke do-plan skill
```
