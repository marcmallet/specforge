---
description: Review a feature implementation against its spec using parallel sub-agents
argument-hint: <ticket-number>
allowed-tools: Read, Write, Bash(mkdir -p *)
model: inherit
---
You are coordinating a feature review using sub-agents.

## Specs directory

Read the project config at: @.claude/specforge/config.json

If the file exists, use the value of the `specsDir` field as the base directory.
If the file does not exist, fall back to `.claude/specforge/` inside the current project.

The full path for this ticket is: <specs-dir>/$ARGUMENTS/

Read the spec at: <specs-dir>/$ARGUMENTS/spec.md
Read the plan at: <specs-dir>/$ARGUMENTS/plan.md

## Agent Detection

Before spawning any agents, check for project-level agents in `.claude/agents/`:

- If `.claude/agents/spec-coverage.md` exists → use it for spec coverage review, otherwise use the plugin's `agents/spec-coverage.md`
- If `.claude/agents/code-quality.md` exists → use it for code quality review, otherwise use the plugin's `agents/code-quality.md`

## Run both agents in parallel at the same time

### Spec Coverage Agent
Pass it:
- The full spec
- All implemented files

### Code Quality Agent
Pass it:
- The full spec
- All implemented files

## After both agents finish
1. Merge findings into a single report saved to <specs-dir>/$ARGUMENTS/review.md
2. Note which agents were used (project or plugin) in the report
3. Suggest any additions to ~/.claude/CLAUDE.md but ask for confirmation before writing them
4. Suggest the next step: "Check the full status with `/specforge:status $ARGUMENTS`"
