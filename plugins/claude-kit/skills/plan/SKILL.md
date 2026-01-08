---
name: plan
description: Plan implementation tasks with parallel exploration and user approval. Use when user says "plan", "let's plan", "help me plan", or runs /plan command.
---

# Plan

Plan implementation tasks with parallel exploration, architecture design, and user approval.

## Plan Workflow

### Phase 1: Initial Understanding

**Goal:** Gain comprehensive understanding of the user's request by reading code and asking questions.

1. Focus on understanding the user's request and the code associated with their request

2. **Launch up to 3 Explore agents IN PARALLEL** (single message, multiple tool calls):
   - Use 1 agent when task is isolated to known files or user provided specific paths
   - Use multiple agents when: scope is uncertain, multiple areas involved, or need to understand patterns
   - Quality over quantity - 3 agents maximum

   Example focuses for parallel agents:
   - Agent 1: Search for existing implementations
   - Agent 2: Explore related components
   - Agent 3: Investigate testing patterns

3. After exploring, use **AskUserQuestion** to clarify ambiguities upfront

### Phase 2: Design

**Goal:** Design an implementation approach.

Launch **Plan agent(s)** to design implementation based on exploration results.

| Scenario | Agents |
|----------|--------|
| Simple task | 1 Plan agent |
| Complex/multi-area | 2-3 Plan agents with different perspectives |
| Trivial (typo, single-line) | Skip agents entirely |

**Example perspectives:**
- New feature: simplicity vs performance vs maintainability
- Bug fix: root cause vs workaround vs prevention
- Refactoring: minimal change vs clean architecture

In the agent prompt:
- Provide comprehensive background from Phase 1 (filenames, code paths)
- Describe requirements and constraints
- Request detailed implementation plan

### Phase 3: Review

**Goal:** Review plans and ensure alignment with user's intentions.

1. Read critical files identified by agents to deepen understanding
2. Ensure plans align with user's original request
3. Use **AskUserQuestion** to clarify any remaining questions

### Phase 4: Final Plan

**Goal:** Write your final plan to the plan file.

Write plan to `.claude/plans/[plan-name].md`:

```markdown
# Plan: [Name]

## Summary
[1-2 sentence overview]

## Steps

### Step 1: [Description]
- **Files:** [files to modify]
- **Pattern:** [reference similar code at path/to/example.ts]
- **Dependencies:** [what this step needs from other steps]

### Step 2: [Description]
...

## Verification
[How to test the changes end-to-end]
```

**Plan file requirements:**
- Include only your recommended approach (not all alternatives)
- Concise enough to scan quickly, detailed enough to execute
- Include paths of critical files to modify
- Include verification section (how to test)

### Phase 5: Exit Plan Mode

Call **ExitPlanMode** to request user approval.

**Important:**
- Use AskUserQuestion to clarify requirements/approach
- Use ExitPlanMode to request plan approval
- Do NOT use AskUserQuestion to ask "Is this plan okay?" - that's what ExitPlanMode does

---

## Plan Quality Checklist

Before finalizing, ensure each step has:

| Required | Description |
|----------|-------------|
| Target file path | Exact file(s) to modify |
| Pattern/example | Similar code to follow (or CLAUDE.md reference) |
| Dependencies | What this step needs (data, files, env vars) |

## Pre-flight Checks

Before finalizing the plan:

1. **Config verification** - Use Grep to verify config files have required keys
2. **Env vars** - Use Bash to verify env vars exist: `test -n "$VAR_NAME"`
3. **Patterns** - Read 1-2 similar implementations per step type

If anything missing → clarify with user, don't finalize incomplete plan.

---

## After Approval

Once user approves, invoke the **do-plan** skill to execute:
- Say "do the plan" or "execute"
- Or run `/do-plan`
