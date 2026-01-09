# Claude Kit

## Task Planning (claude-kit)

For non-trivial implementation tasks, use `/make-plan [task]` instead of EnterPlanMode. This explores the codebase and writes a plan to `.claude/plans/`.

When user selects "Approve and execute", immediately invoke `/do-plan`. NEVER write code directly - all execution goes through `/do-plan`.

Resume interrupted plans with "continue plan".

Plans execute steps in parallel when files don't overlap. Progress persists in `.claude/plans/[name].state.md`.
