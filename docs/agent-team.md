# Agent Team: Mona's Project Pulse Dashboard

Four custom agents in `.github/agents/` build the dashboard. You talk to the Orchestrator; it delegates to the specialists.

| Agent | Role | Model | Tools |
|-------|------|-------|-------|
| **Orchestrator** | Breaks requests into phases, assigns file scopes, delegates, verifies the integrated result. Does not implement. | Claude Opus 5.5 | read, agent, memory |
| **Planner** | Researches the repo and docs, then returns a plan: steps, file assignments, dependencies, parallel vs. sequential work, edge cases, validation, open questions. Writes no code. | Claude Opus 5.5 | read, search, web, memory, todo |
| **Coder** | Implements logic and fixes bugs within its assigned files. Also creates support files such as `.vscode/launch.json` (strict JSON, `cwd` = `${workspaceFolder}/app`, opens `index.html`). | GPT-5.5 | read, edit, search, execute, web, memory, todo |
| **Designer** | Handles UI/UX, accessibility, and visual design. Produces a polished dashboard with project cards, status badges, priority treatment, responsive layout, and `.dashboard` / `.project-card` CSS hooks. | Gemini 3.7 Flash | read, edit, search, web, memory, todo |

## Workflow

1. The Orchestrator asks the Planner for a plan.
2. It splits the plan into phases with explicit file scopes.
3. Tasks run in parallel only when file scopes don't overlap and there are no data dependencies. Otherwise they run sequentially.
4. The Orchestrator checks that the pieces fit together and reports the outcome.

## Shared rules

- Stay within the assigned file scope.
- No agent stages, commits, or pushes. You control git through Copilot CLI prompts.
