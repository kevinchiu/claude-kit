---
name: plan
description: Plan implementation tasks with parallel exploration and explicit user approval. Trigger: "plan", "let's plan", "help me plan", /plan command.
---

# Plan Skill

Custom plan mode using AskUserQuestion for approval. Does NOT use EnterPlanMode/ExitPlanMode.

## Phases

1. EXPLORE - Launch parallel Explore agents
2. CLARIFY - AskUserQuestion to resolve ambiguities
3. DESIGN - Launch Plan agent(s)
4. WRITE - Write plan to .claude/plans/[name].md
5. PRE-FLIGHT - Validate everything for zero-prompt execution
6. APPROVAL - AskUserQuestion for explicit approval

## Phase 1: Explore

Launch 6 Explore agents in ONE message. All use: subagent_type: Explore, model: sonnet.

Tools: Glob, Grep, Read, Bash, LSP. Also use any other available tools (MCP, host tools like tree, etc.).

| Agent | Prompt | Report |
|-------|--------|--------|
| structure | Map dirs. Glob source files. Read entry points. git log --oneline -20 for recent activity. | tree, key dirs, entry points, active areas |
| patterns | Find patterns for [AREA]. Grep, LSP find-references. git blame to understand intent. | locations, code examples, conventions, why |
| dependencies | Analyze deps for [FILES]. Grep, LSP go-to-definition. npm ls or pip list for dep tree. | dep graph, modification order, conflicts |
| types | Find types/interfaces for [AREA]. Grep for type/interface defs. Read type files. LSP hover. | types, data shapes, API contracts |
| tests | Find test patterns for [AREA]. Glob for test files. Read examples. Locate fixtures, mocks. git log. | test locations, conventions, fixtures |
| config | Analyze build/config. Read package.json, tsconfig, .env.example. Grep for process.env/import.meta.env. | build process, env vars, external deps |

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
- Pattern/example reference

## Phase 5: Pre-flight Validation

Resolve all blockers now. Try to fix automatically first - system prompts user for permissions when needed.

| Issue | Resolution |
|-------|------------|
| Pattern file missing | Find alternative, update plan |
| Target dir missing | Create it |
| Env var missing | Add to .env.example, prompt user to set |
| Dependency missing | Install it (npm install, pip install, etc.) |
| File conflict | Check git status, resolve or note in plan |
| Config key missing | Add with sensible default |

After resolving, output summary:
```
Pre-flight: [N] resolved, [M] need user action
[List what was resolved]
[List what needs user action]
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

## Tool Priority

| Task | Use | Avoid |
|------|-----|-------|
| Read file | Read | cat, head, tail |
| Search | Grep | grep, rg |
| Find files | Glob | find, ls |
| Edit | Edit | sed, awk |
| Write | Write | echo, heredoc |
