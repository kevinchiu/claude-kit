---
name: parallel-explore
description: Parallel codebase exploration with 6 specialized agents. Returns structured context for planning and execution.
---

# Parallel Explore Skill

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
