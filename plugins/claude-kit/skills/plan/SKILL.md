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

Tools: Glob, Grep, Read, LSP (go-to-definition, find-references, hover).

| Agent | Prompt | Report |
|-------|--------|--------|
| structure | Map dirs. Glob source files. Read entry points. | tree, key dirs, entry points |
| patterns | Find patterns for [AREA]. Grep, LSP find-references. | locations, code examples, conventions |
| dependencies | Analyze deps for [FILES]. Grep, LSP go-to-definition. | dep graph, modification order, conflicts |
| types | Find types/interfaces for [AREA]. LSP hover. | types, data shapes, API contracts |
| tests | Find test patterns for [AREA]. Locate fixtures, mocks. | test locations, conventions, fixtures |
| config | Analyze build/config. Read package.json, tsconfig, .env.example. | build process, env vars, external deps |

## Phase 2: Clarify

Use AskUserQuestion for:
- Unclear requirements
- Multiple valid approaches
- Scope boundaries
- Technical preferences

Skip if requirements are unambiguous.

## Phase 3: Design

Launch up to 3 Plan agents (Opus) based on complexity:

| Complexity | Agents | Perspectives |
|------------|--------|--------------|
| Simple | 1 | Single approach |
| Medium | 2 | Minimal vs thorough |
| Complex | 3 | Simplicity vs performance vs maintainability |

**Task (design):**
- subagent_type: Plan
- model: opus
- prompt: Design implementation for [REQUEST] from [PERSPECTIVE] perspective. Context: [exploration results]. Clarifications: [Phase 2 answers]. Provide: steps, files per step, dependencies, risks.

## Phase 4: Write Plan

Write to `.claude/plans/[plan-name].md`:

```
# Plan: [Name]

## Summary
[1-2 sentences]

## Steps

### Step 1: [Description]
- Files: [exact paths]
- Pattern: [path/to/similar/code.ts]
- Dependencies: None
- Details: [implementation notes]

### Step 2: [Description]
- Files: [exact paths]
- Pattern: [path/to/similar/code.ts]
- Dependencies: Step 1
- Details: [implementation notes]

## Verification
- [How to verify each step]
- [End-to-end test command]
```

Required for each step:
- Exact file paths
- Pattern/example reference
- Explicit dependencies

## Phase 5: Pre-flight Validation

Verify everything for zero-prompt execution. Resolve ALL blockers now.

### Checks

| Check | Verify | If Missing |
|-------|--------|------------|
| Pattern files exist | Read each pattern | AskUserQuestion for alternative |
| Target dirs exist | Glob parent dirs | Create or ask |
| Env vars set | Bash: test -n "$VAR" | AskUserQuestion |
| Config keys present | Grep config files | AskUserQuestion |
| Dependencies installed | Bash: which tool | AskUserQuestion |
| No file conflicts | Glob for existing | AskUserQuestion: overwrite? |

### Decisions

For each step verify:
- Exact approach specified (no "could do X or Y")
- Naming conventions defined
- Error handling pattern specified
- Test scope defined

Any ambiguity found: AskUserQuestion to resolve.

### External Services

If plan uses external services:
- Verify API base URL configured
- Verify connection strings exist
- Verify credentials/tokens available

### Report

After validation, output summary:
```
Pre-flight: [N] checks passed, [M] issues
[List any resolved issues]
Ready for approval.
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
