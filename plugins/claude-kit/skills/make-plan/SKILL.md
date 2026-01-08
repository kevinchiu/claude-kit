---
name: make-plan
description: Plan implementation tasks with parallel exploration and explicit user approval. Trigger: "make a plan", "let's plan", "help me plan", /make-plan command.
---

# Make-Plan Skill

Custom plan mode using AskUserQuestion for approval. Does NOT use EnterPlanMode/ExitPlanMode.

## Phases

1. EXPLORE - Launch parallel Explore agents
2. CLARIFY - AskUserQuestion to resolve ambiguities
3. DESIGN - Launch Plan agent(s)
4. WRITE - Write plan to .claude/plans/[name].md
5. PRE-FLIGHT - Validate everything for zero-prompt execution
6. APPROVAL - AskUserQuestion for explicit approval

## Phase 1: Explore

Invoke the parallel-explore skill to launch 6 agents and gather codebase context.

## Phase 2: Clarify

Use AskUserQuestion for:
- Unclear requirements
- Multiple valid approaches
- Scope boundaries
- Technical preferences

Skip if requirements are unambiguous.

## Phase 3: Design

Default: 1 Plan agent (opus). Follow Claude Code's built-in principles (minimal changes, no over-engineering, no premature abstraction).

Multiple agents only for genuine architectural tradeoffs:

| Tradeoff | Perspectives |
|----------|--------------|
| Build vs buy | "Use existing lib" vs "Build custom" |
| Scope | "Minimal (YAGNI)" vs "Extensible" |
| Performance critical | "Simple first" vs "Optimized" |

**Task (design):**
- subagent_type: Plan
- model: opus
- prompt: |
    Design implementation for [REQUEST].

    Exploration context:
    - Structure: [summary from structure agent]
    - Patterns: [summary from patterns agent]
    - Dependencies: [summary from dependencies agent]
    - Types: [summary from types agent]
    - Tests: [summary from tests agent]
    - Config: [summary from config agent]

    Clarifications: [Phase 2 answers]

    Provide: steps, files per step, dependencies between steps, risks.

## Phase 4: Write Plan

Write to `.claude/plans/[plan-name].md`:

```
# Plan: [Name]

## Summary
[1-2 sentences]

## Exploration Context
- Structure: [key dirs, entry points]
- Patterns: [relevant patterns found, example paths]
- Dependencies: [dep graph relevant to this plan]
- Types: [key types/interfaces involved]
- Tests: [test patterns, fixture locations]
- Config: [env vars needed, build considerations]

## Steps

### Step 1: [Description]
- Files: [exact paths]
- Dependencies: None
- Pattern: [path/to/similar/code.ts]
- Notes: [implementation details, edge cases, decisions]

### Step 2: [Description]
- Files: [exact paths]
- Dependencies: Step 1
- Pattern: [path/to/similar/code.ts]
- Notes: [implementation details, edge cases, decisions]

## Verification
[End-to-end test command]
```

Required for each step:
- Exact file paths
- Explicit dependencies
- Pattern reference

## Phase 5: Pre-flight Validation

Detect blockers and get user input where needed. No side effects - execution is do-plan's job.

Ask user only for: login credentials or API keys.

Auto-resolved by do-plan (don't ask):
- Target dir missing (mkdir)
- Dependency missing (npm/pip install)
- Config key missing (add default)

After validation:
```
Pre-flight complete.
[List items that need user action before proceeding]
```

## Phase 6: Approval

Use AskUserQuestion:
- question: "Plan written to .claude/plans/[name].md. Ready to execute?"
- options:
  - "Approve and execute" - Start do-plan skill
  - "Review first" - Wait for user
  - "Revise plan" - Get feedback, update, re-submit

Response handling:

| Response | Action |
|----------|--------|
| Approve | Invoke do-plan skill |
| Review | Wait for user response |
| Revise | Ask for feedback, update plan, re-submit |
| Other | Treat as revision feedback |

## After Approval

Invoke do-plan: say "Do the plan" or run /do-plan [plan-file]

