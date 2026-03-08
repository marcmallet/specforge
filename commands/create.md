---
description: Create a new feature spec from a ticket number, title and description
argument-hint: <ticket-number> | <title> | <description>
allowed-tools: Read, Write, Bash(mkdir -p *)
model: inherit
---
You are helping create a feature spec from a brief description.

The user input is: $ARGUMENTS

The input format is: <ticket-number> | <short title> | <description>

Parse it like this:
- Ticket number: the first part e.g. "PROJ-1234"
- Title: the second part e.g. "Update header navigation"
- Description: the third part e.g. "We need to update the header navigation background color from white to black"

## Specs directory

Read the project config at: @.claude/specforge/config.json

If the file exists, use the value of the `specsDir` field as the base directory.
If the file does not exist, fall back to `.claude/specforge/` inside the current project.

The full path for this ticket's spec is: <specs-dir>/<ticket-number>/spec.md

Create the ticket directory if it does not already exist.

## Spec template

Using the parsed description, generate a complete feature spec:

# Feature: <title>
**Ticket:** <ticket-number>

## Overview
2-3 sentences describing what this feature is and why it exists.

## Requirements
- [ ] (list each discrete requirement)

## Acceptance Criteria
- (list each testable condition that proves the feature is done)

## Technical Notes
- (any implementation hints, libraries, patterns to use)

## Out of Scope
- (explicitly list what this feature does NOT cover)

Rules:
1. Infer reasonable requirements from the description — don't be too literal
2. Make acceptance criteria testable and specific
3. Flag anything you assumed with a note like "(assumed)"
4. Do NOT save the file yet — show the spec to the user first and ask:
   - "Does this look right? Any changes before I save it?"
   - Only save after confirmation
5. After saving, suggest the next step: "Ready to plan? Run `/specforge:plan <ticket-number>`"
