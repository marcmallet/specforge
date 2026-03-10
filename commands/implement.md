---
description: Implement a feature using phased parallel sub-agents for backend, frontend and shared
argument-hint: <ticket-number>
allowed-tools: Read, Write, Edit, Bash(mkdir -p *)
model: inherit
---
You are coordinating the implementation of a feature using sub-agents.

## Specs directory

Read the project config at: @.claude/specforge/config.json

If the file exists, use the value of the `specsDir` field as the base directory.
If the file does not exist, fall back to `.claude/specforge/` inside the current project.

The full path for this ticket is: <specs-dir>/$ARGUMENTS/

Read the spec at: <specs-dir>/$ARGUMENTS/spec.md
Read the plan at: <specs-dir>/$ARGUMENTS/plan.md (if it does not exist, generate the plan first before implementing)

## Agent Detection

Before spawning any agents, check for project-level agents in `.claude/agents/`:

- If `.claude/agents/backend.md` exists → use it for backend tasks, otherwise use the plugin's `agents/backend.md`
- If `.claude/agents/frontend.md` exists → use it for frontend tasks, otherwise use the plugin's `agents/frontend.md`
- If `.claude/agents/shared.md` exists → use it for shared tasks, otherwise use the plugin's `agents/shared.md`

## Phase 1 — Run in parallel

Spawn the backend and frontend agents at the same time. Do not wait for one before starting the other.

### Backend Agent
Pass it:
- The full spec
- The full plan
- Only the tasks tagged [backend]
- The Shared Contracts section from the plan

### Frontend Agent
Pass it:
- The full spec
- The full plan
- Only the tasks tagged [frontend]
- The Shared Contracts section from the plan

## Phase 2 — Run after Phase 1 completes

Wait for BOTH backend and frontend agents to finish before starting.

### Shared Agent
Pass it:
- The full spec
- The full plan
- Only the tasks tagged [both]
- The Shared Contracts section from the plan
- All files created or modified by the backend and frontend agents

## After all agents finish
1. Update the checklist in <specs-dir>/$ARGUMENTS/plan.md marking all completed tasks as done
2. If any agent reports an ambiguity, stop and ask the user before proceeding
3. Do not implement anything outside the spec scope
4. Summarise what was built, which agents were used (project or plugin), and list any deviations from the spec
5. Remind the user to write tests for the implemented code
6. Suggest the next step: "Ready to review? Run `/specforge:review $ARGUMENTS`"
