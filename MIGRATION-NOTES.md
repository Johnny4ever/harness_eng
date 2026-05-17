# Migration Notes: v0 → v1

This document records what changed in the Phase 7 atomic switch and how to adapt any projects using the previous architecture.

## What Changed

### Agents: 29 → 4

The previous architecture had 29 agents across three groups (Project Intelligence, BI Agents, Resource Admin). These have been replaced by 4 generic agents:

| Old | Replaced by |
|---|---|
| `project-dispatcher`, `project-synthesizer`, `project-strategist`, `project-reviewer` | `planner` |
| `bi-orchestrator` + 14 BI step agents (`bi-requirement-intake`, `bi-kpi-metric-definition`, etc.) | `generator` + skills |
| `resource-orchestrator` + 5 resource agents | `collector` + collect/* skills |
| *(no equivalent)* | `evaluator` (new — implements sprint contracts and hard-threshold grading) |

The old agent files were deleted in the Phase 7 commit. They are preserved at the git tag `v0-domain-specific` if you need to reference them.

### `/bi_agent` command removed

The `/bi_agent` slash command has been deleted. Use `/project` instead — the planner classifies your intent and selects the appropriate playbook (`bi-dashboard`, `dbt-data-product`, etc.) automatically.

### Skills replace embedded agent logic

Domain-specific work that previously lived in individual BI step agents now lives in composable Skills. The generator loads them from `SKILLS-CATALOG.md` on demand — you do not invoke skills directly.

### Evaluator is now mandatory

The previous architecture had no evaluation layer. In v1, every checkpoint goes through:
1. Sprint contract (written by evaluator before generation starts)
2. Hard-threshold grading (PASS/FAIL only, no partial credit)
3. Feedback synthesis if FAIL (specific, actionable rework brief)

This adds one model invocation per checkpoint iteration but prevents partial or placeholder work from advancing.

### Output structure is playbook-defined

Previously the BI pipeline used a fixed 14-step output layout. Now each playbook defines its own `output_root` and step folders. The `bi-dashboard` playbook is functionally equivalent to the old BI pipeline.

## If You Have Existing Projects

Projects started with the v0 architecture can continue to completion using the v0 files at the `v0-domain-specific` tag. Do not mix v0 agent invocations with v1 files.

To migrate a mid-flight project to v1:
1. Check out the current project state
2. Map existing output files to the v1 output structure (step folders are equivalent: 01-requirement → step 01, etc.)
3. Create a `project-journal.md` for the project under `docs/projects/<slug>/`
4. Use `/project` from that point forward

## Rollback

If you need to roll back to v0:
```
git checkout v0-domain-specific -- .claude/
```
This restores all 29 old agent files and the `/bi_agent` command.
