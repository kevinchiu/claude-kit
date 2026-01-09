---
name: parallel-explore
description: Parallel codebase exploration with 6 specialized agents. Returns structured context for planning and execution.
---

# Parallel Explore Skill

Launch 6 Explore agents in ONE message (subagent_type: Explore, model: sonnet):

1. **structure** - directory layout, entry points, recent activity
2. **patterns** - relevant code patterns and conventions
3. **dependencies** - dependency graph, modification order
4. **types** - types/interfaces involved
5. **tests** - test patterns, fixtures, conventions
6. **config** - build process, env vars, external deps

## Output Format

Return structured context:

```
## Exploration Context
- Structure: [key dirs, entry points]
- Patterns: [relevant patterns found, example paths]
- Dependencies: [dep graph relevant to task]
- Types: [key types/interfaces involved]
- Tests: [test patterns, fixture locations]
- Config: [env vars needed, build considerations]
```
