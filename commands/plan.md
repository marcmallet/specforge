---
description: Generate a tagged implementation plan from a feature spec
argument-hint: <ticket-number>
allowed-tools: Read, Write, Bash(mkdir -p *)
model: inherit
---
You are helping plan implementation of a feature spec.

## Specs directory

Read the project config at: @.claude/specforge/config.json

If the file exists, use the value of the `specsDir` field as the base directory.
If the file does not exist, fall back to `.claude/specforge/` inside the current project.

The full path for this ticket is: <specs-dir>/$ARGUMENTS/

Read the feature spec at: <specs-dir>/$ARGUMENTS/spec.md

## Plan

Then do the following:
1. Summarize the feature in 2-3 sentences
2. Identify all files that will need to be created or modified in the current project
3. Break the work into ordered tasks as a markdown checklist, each small enough to implement and test independently
4. Tag every task with one of:
   - `[backend]` — server side only, no frontend files touched
   - `[frontend]` — client side only, no backend files touched
   - `[both]` — spans both backend and frontend (e.g. a template that needs data bindings AND markup)
5. For any task tagged `[both]`, define the shared contract explicitly:
   - File(s) involved
   - Field names, endpoint paths, data shapes both sides must agree on
6. Summarise all shared contracts in a ## Shared Contracts section at the bottom of the plan
7. Flag any ambiguities, missing information, or decisions that need to be made before starting
8. Estimate complexity: Low / Medium / High and explain why
9. Save the final plan to <specs-dir>/$ARGUMENTS/plan.md

Do not write any code yet. This is planning only.

After saving the plan, suggest the next step: "Ready to implement? Run `/specforge:implement $ARGUMENTS`"
