# specforge

A Claude Code plugin for spec-driven feature development. Write a one-liner, get a full spec, plan, implementation and review — all tracked in one place.

## Commands

| Command | Description |
|---|---|
| `/specforge:config` | Set the directory where specs, plans and reviews are saved for this project |
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

### 1. Create a spec — `/specforge:create`

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

A good plan prevents wasted effort. This command reads your spec and breaks the work into a prioritised, tagged checklist of tasks. Each task is tagged as `[backend]`, `[frontend]`, `[both]` or `[tests]` so there is zero ambiguity about who does what. Any tasks that span both backend and frontend produce a Shared Contracts section — field names, endpoint paths and data shapes that both sides must agree on before implementation starts. This is also where complexity is estimated and ambiguities are flagged before any code is written.

```bash
/specforge:plan PROJ-1234
```

The plan is saved to `<specs-dir>/PROJ-1234/plan.md`.

---

### 3. Implement the feature — `/specforge:implement`

Implementation runs in three phases using parallel sub-agents to avoid conflicts and maximise speed. In Phase 1, the backend and frontend agents work simultaneously on their respective tasks. In Phase 2, the shared agent handles anything that spans both sides, with full context of what was already built. In Phase 3, the tests agent writes coverage for everything. The plan checklist is updated as each task completes so you always know where things stand.

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

## Agents

Specforge ships with generic agents that work for any project out of the box:

| Agent | Role |
|---|---|
| `backend` | Controllers, models, migrations, routes |
| `frontend` | Templates, styles, JavaScript |
| `shared` | Files that span both backend and frontend e.g. templates needing data bindings and markup |
| `tests` | Unit and integration tests |
| `spec-coverage` | Verifies every requirement was implemented |
| `code-quality` | Reviews for errors, security, consistency |

The implement command runs them in phases:

```
Phase 1: backend + frontend  (parallel)
Phase 2: shared              (after Phase 1)
Phase 3: tests               (after Phase 2)
```

### Customising Agents

Specforge automatically detects project-level agents before falling back to the bundled ones. If you create any of the following in your project, they will be used instead:

```
your-project/
  .claude/
    agents/
      backend.md        ← overrides bundled backend agent
      frontend.md       ← overrides bundled frontend agent
      shared.md         ← overrides bundled shared agent
      tests.md          ← overrides bundled tests agent
      spec-coverage.md  ← overrides bundled spec-coverage agent
      code-quality.md   ← overrides bundled code-quality agent
```

You only need to create the agents you want to customise — any agent not found in your project falls back to the bundled version automatically.

For stack-wide conventions that apply across all agents (coding standards, framework preferences, file structure), add them to your project's `CLAUDE.md` — all agents will pick these up without needing to be customised.
