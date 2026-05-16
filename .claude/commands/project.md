# Project (`/project`)

You were invoked via the **`/project`** slash command.

Act as **`project-dispatcher`** and follow `rules/project-intelligence-agent.md`.

## Sub-commands

Parse the sub-command and arguments from: `$ARGUMENTS`

If no sub-command is given, **classify intent from the natural language** per the dispatcher's intent classification table.

| Sub-command | Action |
|---|---|
| `<natural language>` | Dispatcher classifies intent and routes automatically |
| `analyze` | Re-synthesize artifact library + produce strategy (uses cache if valid) |
| `analyze --force` | Force re-synthesis even if cache is valid |
| `plan` | Run strategist only (assumes valid synthesis exists) |
| `execute` | Route to executor if an approved strategy exists in the project journal |
| `review` | Review last executor run, produce lesson-learned artifacts |
| `status` | Show current state from project journal |
| `reset` | Reset project journal and invalidate cache for the slug |
| `ingest <url-or-pasted>` | Alias for `/admin_resource ingest` |
| `search <query>` | Alias for `/admin_resource search` |

## First run

If no project journal or cache manifest exists yet, bootstrap them before routing. See first-run bootstrapping section in `project-dispatcher` agent.

## Execution flow

### 1. Act as `project-dispatcher`

- Read `docs/.cache-manifest.md` and `docs/projects/<slug>/project-journal.md` (routing section only).
- If the slug is not yet known, scan `docs/INDEX-by-project.md` (or list `docs/projects/`) to identify the active project, or ask the user.
- Classify intent, determine route and sequence, report routing decision to user in one sentence.

### 2. Route to agents

| Agent | Role |
|---|---|
| `project-synthesizer` | Artifact library → synthesis doc |
| `project-strategist` | Synthesis → plan + journal |
| `project-reviewer` | Executor output → lesson-learned |
| `resource-orchestrator` | Ingestion, search, update |
| `bi-orchestrator` | BI dashboard execution |

### 3. Human gates

Human gates are enforced by downstream agents, not the dispatcher:
- **`project-strategist`** holds for user approval before signalling execution can proceed.
- **`project-reviewer`** holds for user decision when execution quality is `partial` or `failed`.

The dispatcher does not proceed past a gate until the relevant agent signals `approved: true`.

### 4. Keep user informed

After each agent completes, tell the user:
- What just completed and what file was produced
- What happens next
- Whether a human gate is active

## Output structure

All outputs follow `rules/project-intelligence-output-structure.md`:

```
docs/
  .cache-manifest.md
  INDEX-by-project.md
  INDEX-by-type.md
  presentations/
  projects/
    <slug>/
      project-journal.md
      DECISIONS.md
      synthesis/
        synthesis.md
        versions/
      strategy/
        strategy.md
        versions/
      reviews/
        review-iter<N>.md
      output/
        STATUS.md
        01-requirement/ ... 14-governance/
```

Lesson-learned artifacts go to `artifacts/ad-hoc/` following the resource-admin naming convention.

Every deliverable is subject to the **universal versioning protocol** — prior versions archived to `versions/`, one-line entry appended to `DECISIONS.md` on every version bump.

## Integration with other agent groups

- **Resource Admin**: `/project ingest` and `/project search` route to `resource-orchestrator`.
- **BI Agents**: `bi-orchestrator` receives a strategy handoff block from `project-strategist`.

## Context budget

The dispatcher reads at most 2 files. All downstream agents follow the context budgets defined in `rules/project-intelligence-output-structure.md`.
