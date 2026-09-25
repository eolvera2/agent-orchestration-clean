# Agent team

I am using GitHub Copilot CLI in a Codespace to orchestrate a custom agent team to build Mona's Project Pulse dashboard.

## Agents

- **Orchestrator**
  - Model: Claude Opus 4.7 (copilot)
  - Responsibility: Coordinates the Planner, Coder, and Designer agents. Breaks requests into phases, assigns explicit file scopes, decides what runs in parallel vs. sequentially, and reports final outcomes. Does not implement work itself and does not perform git operations.
  - Definition: `.github/agents/orchestrator.agent.md`

- **Planner**
  - Model: Claude Opus 4.7 (copilot)
  - Responsibility: Researches the codebase, docs, dependencies, and edge cases to produce an implementation plan with ordered steps, file assignments, dependencies, parallelizable work, and validation expectations. Does not write code or perform git operations.
  - Definition: `.github/agents/planner.agent.md`

- **Designer**
  - Model: Gemini 3.1 Pro (copilot)
  - Responsibility: Owns UI/UX, accessibility, information architecture, and interaction flow. For Project Pulse, builds a polished dashboard with project cards, status badges, priority treatment, and deterministic CSS hooks (`.dashboard`, `.project-card`). Stays within assigned files and does not perform git operations.
  - Definition: `.github/agents/designer.agent.md`

- **Coder**
  - Model: GPT-5.5 (copilot)
  - Responsibility: Implements code within the file scope assigned by the Orchestrator, with clear structure, explicit errors, and testable behavior. For Project Pulse, also creates supporting run/launch configuration (e.g., `.vscode/launch.json` pointing to `app` and opening `index.html`). Stays within assigned files and does not perform git operations.
  - Definition: `.github/agents/coder.agent.md`

All agents are restricted from staging, committing, or pushing changes — the learner controls all git operations through Copilot CLI prompts.
