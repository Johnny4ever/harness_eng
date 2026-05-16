# BI Agent (`/bi_agent`)

You were invoked via the **`/bi_agent`** slash command.

Act as **`bi-orchestrator`** and follow `rules/bi-agent.md`.

This command invokes `bi-orchestrator` in **fallback mode** — it accepts raw requirement context directly without requiring a strategy doc from the Project Intelligence layer. Use this for direct, single-project BI requests where you do not need the full analyze → plan → execute loop.

For end-to-end project management (ingest → analyze → plan → execute → review), use `/project` instead.

## Usage

```
/bi_agent <customer_requirement_link_or_context>
```

Parse the requirement context from: `$ARGUMENTS`

## What happens

1. `bi-orchestrator` reads the requirement context you provided.
2. Three checkpoints with user review between each:

   | Checkpoint | Name | Key agents | User reviews |
   |---|---|---|---|
   | 1 | Requirement specification | `bi-requirement-intake`, `bi-kpi-metric-definition`, `bi-stakeholder-alignment` | Requirement spec, KPI dictionary, alignment summary |
   | 2 | Data discovery and model | `bi-data-discovery`, `bi-data-quality-profiling`, `bi-semantic-model-design`, `bi-transformation-sql-build` | Source mapping, feasibility, semantic model, curated SQL |
   | 3 | Dashboard wireframe | `bi-wireframe-ux` (uses checkpoint 2 discovery output as primary input) | Wireframe/UX pack with validated sample data, version index |

3. **Post-checkpoint steps** (when user is ready): `bi-build`, `bi-validation-qa`, `bi-release-deployment`, `bi-documentation-knowledge`.

4. **Cross-cutting agents** (invoked when needed at any stage): `bi-source-enablement` (unblock data access), `bi-governance-reuse` (enterprise semantic consistency).

## Output

All deliverables go to `docs/projects/<slug>/output/<step-subfolder>/` following the step-numbered structure in `rules/bi-output-structure.md`.

Keep the user informed which checkpoint you are in and what happens next.
