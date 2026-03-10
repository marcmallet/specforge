<p align="center">
  <img alt="SpecForge" src="/assets/logo.svg" width="300" /><br><br>
  A Claude Code plugin for spec-driven feature development. Write a one-liner, <br>
  get a full spec, plan, implementation and review — all tracked in one place.
</p>

<hr>

## Commands

| Command | Description |
|---|---|
| `/specforge:config` | Set the directory where specs, plans and reviews are saved for this project |
| `/specforge:jira` | Fetch a Jira ticket and create a spec from it automatically (requires MCP) |
| `/specforge:create` | Create a new feature spec from a ticket number, title and description |
| `/specforge:plan` | Generate an implementation plan from a spec |
| `/specforge:implement` | Implement a feature using parallel sub-agents |
| `/specforge:review` | Review implementation against the spec |
| `/specforge:status` | Check the current status of a feature |

## Installation

```bash
/plugin marketplace add marcmallet/specforge
/plugin install specforge@marcmallet/specforge
```

### Updating

```bash
/plugin update specforge@marcmallet/specforge
/reload-plugins
```

## Configuration

All specs, plans and reviews are saved to a single directory, one subfolder per ticket:

```
<specs-dir>/
  PROJ-1234/
    spec.md
    plan.md
    review.md
```

### Setting the specs directory

By default all files are saved inside the current project at `.claude/specforge/`. To save to a different location, run:

```bash
/specforge:config ~/Documents/Features
```

This creates `.claude/specforge/config.json` in your project:

```json
{
  "specsDir": "~/Documents/Features"
}
```

Both `~` and absolute paths work. The config is per-project — each project can point to a different location. If `/specforge:config` has never been run, all files save to `.claude/specforge/` inside the current project.

If you want teammates to use the same location, commit `.claude/specforge/config.json` to version control. If you want it to be personal, add it to `.gitignore`.

## Workflow

### 0. Configure specs location — `/specforge:config` (optional)

By default specs save to `.claude/specforge/` inside your project. If you want to save them somewhere else — like a local folder or a shared team folder — run this once per project before you start:

```bash
/specforge:config ~/Documents/Features
```

You only need to do this once. The path is saved to `.claude/specforge/config.json` and picked up automatically by all other commands.

---

### 1. Create a spec — `/specforge:create` or `/specforge:jira`

There are two ways to create a spec.

**From Jira** — if your team uses Jira and you have the `mcp-atlassian` MCP server configured:

```bash
/specforge:jira PROJ-1234
```

This fetches the ticket title, description and acceptance criteria directly via MCP and generates a spec from it. No copy-pasting required. See [Jira Integration](#jira-integration) for setup.

**Manually** — works without any external tools:

Before writing any code, you need a clear definition of what you're building. The create command takes your plain English description and turns it into a structured spec covering requirements, acceptance criteria, technical notes and out of scope items. Claude shows you the draft first and only saves it after you confirm, so you stay in control of what goes on record.

The command takes three parameters separated by `|`:

| Parameter | Description | Example |
|---|---|---|
| Ticket number | Your issue tracker ticket ID | `PROJ-1234` |
| Title | Short feature name | `Update header navigation` |
| Description | What needs to be built and why | `Change background color from white to black` |

```bash
/specforge:create PROJ-1234 | Update header navigation | Background color should change from white to black
```

The spec is saved to `<specs-dir>/PROJ-1234/spec.md`.

---

### 2. Plan the feature — `/specforge:plan`

A good plan prevents wasted effort. This command reads your spec and breaks the work into a prioritised, tagged checklist of tasks. Each task is tagged as `[backend]`, `[frontend]`, or `[both]` so there is zero ambiguity about who does what. Any tasks that span both backend and frontend produce a Shared Contracts section — field names, endpoint paths and data shapes that both sides must agree on before implementation starts. This is also where complexity is estimated and ambiguities are flagged before any code is written.

```bash
/specforge:plan PROJ-1234
```

The plan is saved to `<specs-dir>/PROJ-1234/plan.md`.

---

### 3. Implement the feature — `/specforge:implement`

Implementation runs in three phases using parallel sub-agents to avoid conflicts and maximise speed. In Phase 1, the backend and frontend agents work simultaneously on their respective tasks. In Phase 2, the shared agent handles anything that spans both sides, with full context of what was already built. The plan checklist is updated as each task completes so you always know where things stand.

```bash
/specforge:implement PROJ-1234
```

---

### 4. Review the implementation — `/specforge:review`

Once implementation is done, two review agents run in parallel. The spec coverage agent checks every requirement and acceptance criterion against the code, marking each as done, partial or missing, and flags any scope creep. The code quality agent looks for error handling gaps, security concerns and consistency issues. Their findings are merged into a single review report, giving you a clear picture of what shipped versus what was specified.

```bash
/specforge:review PROJ-1234
```

The review is saved to `<specs-dir>/PROJ-1234/review.md`.

---

### 5. Check status — `/specforge:status`

At any point in the workflow you can check where a feature stands. This command reads the spec, plan and review files and gives you a quick summary of task progress broken down by type, review coverage if a review exists, and the next recommended action.

```bash
/specforge:status PROJ-1234
```

---

## File Structure Per Ticket

```
<specs-dir>/
  PROJ-1234/
    spec.md      ← what to build (source of truth)
    plan.md      ← how to build it (tagged checklist)
    review.md    ← what was verified
```

Where `<specs-dir>` is configured via `/specforge:config`, or `.claude/specforge/` inside your project if not set.

## Jira Integration

> **Experimental:** This feature has not yet been tested in production. Use with caution and report any issues.
>
> **This feature is optional.** Specforge works perfectly without it — use `/specforge:create` instead. Only set this up if you actively want to pull tickets directly from Jira.
>
> **Security notice:** Connecting any AI tool to Jira carries risk. The MCP server will act with your account's permissions. Be cautious, use the least-privileged option available to you, and never connect an account with admin access.

The `/specforge:jira` command supports two MCP servers. It will automatically use whichever one you have configured, preferring the official Atlassian server if both are present.

---

### Option A — Official Atlassian Rovo MCP (recommended)

Uses OAuth — no API token stored anywhere. Authenticates via browser using your Atlassian account.

**Caution:** The official server exposes all Jira tools (create, edit, transition, etc.), not just read. Specforge only calls `getJiraIssue`, but the full surface is available to Claude Code. Only use this if you are comfortable with that.

**Setup:**

```bash
claude mcp add --transport sse atlassian https://mcp.atlassian.com/v1/mcp
```

Restart Claude Code. A browser window will open to complete OAuth. Once authenticated, verify with `/mcp` — you should see `atlassian` listed.

Official docs: https://support.atlassian.com/atlassian-rovo-mcp-server/docs/getting-started-with-the-atlassian-remote-mcp-server/

---

### Option B — Community mcp-atlassian server

Uses an API token with two layers of restriction: `ENABLED_TOOLS` limits Claude Code to only `jira_get_issue`, and `READ_ONLY_MODE` blocks all write operations at the server level as a hard guarantee.

**Setup:**

1. Install `uv` if not already available: https://docs.astral.sh/uv/getting-started/installation/

2. Create an Atlassian API token at https://id.atlassian.com/manage-profile/security/api-tokens. Name it something like `specforge-readonly`.

3. Add to your `~/.claude/settings.json`:

```json
{
  "mcpServers": {
    "mcp-atlassian": {
      "command": "uvx",
      "args": ["mcp-atlassian"],
      "env": {
        "JIRA_URL": "https://your-company.atlassian.net",
        "JIRA_USERNAME": "your.email@company.com",
        "JIRA_API_TOKEN": "your_api_token",
        "ENABLED_TOOLS": "jira_get_issue",
        "READ_ONLY_MODE": "true"
      }
    }
  }
}
```

`ENABLED_TOOLS` restricts Claude Code to only the issue read tool. `READ_ONLY_MODE` adds a second independent layer — it blocks all write operations at the server level regardless of any other setting. Together they give the strongest possible read-only guarantee.

4. Restart Claude Code and verify with `/mcp` — you should see `mcp-atlassian` listed with only `jira_get_issue`.

Community server repo: https://github.com/sooperset/mcp-atlassian

---

### Usage

Once either option is configured:

```bash
/specforge:jira PROJ-1234
```

Specforge fetches the ticket, shows you what it found, and gives you a ready-to-run `/specforge:create` command with the details filled in.

## Agents

SpecForge ships with generic agents that work for any project out of the box:

| Agent | Role |
|---|---|
| `backend` | Controllers, models, migrations, routes |
| `frontend` | Templates, styles, JavaScript |
| `shared` | Files that span both backend and frontend e.g. templates needing data bindings and markup |
| `spec-coverage` | Verifies every requirement was implemented |
| `code-quality` | Reviews for errors, security, consistency |

The implement command runs them in phases:

```
Phase 1: backend + frontend  (parallel)
Phase 2: shared              (after Phase 1)
```

### Customising Agents

SpecForge automatically detects project-level agents before falling back to the bundled ones. If you create any of the following in your project, they will be used instead:

```
your-project/
  .claude/
    agents/
      backend.md        ← overrides bundled backend agent
      frontend.md       ← overrides bundled frontend agent
      shared.md         ← overrides bundled shared agent
      spec-coverage.md  ← overrides bundled spec-coverage agent
      code-quality.md   ← overrides bundled code-quality agent
```

You only need to create the agents you want to customise — any agent not found in your project falls back to the bundled version automatically.

For stack-wide conventions that apply across all agents (coding standards, framework preferences, file structure), add them to your project's `CLAUDE.md` — all agents will pick these up without needing to be customised.
