---
description: Set the directory where specs, plans and reviews are saved for this project
argument-hint: <path>
allowed-tools: Read, Write, Bash(mkdir -p *)
model: inherit
---
You are setting the specs directory for this project.

The user has provided: $ARGUMENTS

If no path was provided, ask the user for one before continuing.

Accepted formats:
- Absolute path: `/Users/username/Documents/Features`
- Path with tilde: `~/Documents/Features`
- Relative to project: `./specs`

Steps:
1. Confirm the path with the user before saving
2. Create the directory `.claude/specforge/` in the current project if it does not exist
3. Save the config to `.claude/specforge/config.json` in this format:

```json
{
  "specsDir": "<path>"
}
```

4. Confirm to the user that the config has been saved and that all SpecForge commands in this project will now save to that location
5. Suggest they add `.claude/specforge/config.json` to version control if they want teammates to use the same location
