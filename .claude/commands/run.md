# Run (`/run`)

You were invoked via the **`/run`** slash command.

Act as **`generator`** using the specified playbook. This is a power-user escape hatch that skips the planner. Use `/project` for full lifecycle with planning, synthesis, and strategy.

## Usage

```
/run <playbook-name> <context-or-requirement>
```

Parse the playbook name and context from: `$ARGUMENTS`

**Available playbooks:**
- `kpi-proof` — single-step KPI dictionary proof of concept (Phase 3 test)
- `bi-dashboard` — full 14-step BI dashboard delivery *(Phase 4, pending)*
- `dbt-data-product` — dbt model pipeline *(Phase 6, pending)*
- `analysis-deep-dive` — exploratory analysis *(Phase 4, pending)*

## What happens

1. Read `.claude/playbooks/<playbook-name>.md`
2. Read `.claude/skills/SKILLS-CATALOG.md`
3. Create `docs/projects/<slug>/output/STATUS.md` if it does not exist
4. Signal `evaluator` to write the sprint contract for checkpoint 1
5. Execute steps per the playbook sequence, loading skills from the catalog
6. Hand off to `evaluator` after each checkpoint for grading
7. Apply human gate on PASS — wait for user approval before advancing
8. Run retrospective after all checkpoints complete

## Slug Derivation

If no project journal exists, derive the slug from the context:
- "Sales dashboard Q2 2026" → `sales-dashboard-q2-2026`
- "Customer churn analysis" → `customer-churn-analysis`
- Confirm slug with user before writing any files

## Relationship to `/project`

| | `/project` | `/run` |
|---|---|---|
| Planner involved | ✅ Yes | ❌ No |
| Synthesis phase | ✅ Yes | ❌ No |
| Playbook auto-selected | ✅ Yes | ❌ Manual |
| Evaluator loop | ✅ Yes | ✅ Yes |
| Human gates | ✅ Yes | ✅ Yes |
| Recommended for | Most work | Testing, explicit overrides, pre-specified requirements |
