# Output Structure

Canonical folder layout for all harness outputs. Agents must write to these paths exactly — no improvisation.

## Project Output Tree

```
docs/
  .cache-manifest.md              ← planner: synthesis cache validity flag
  INDEX-by-project.md             ← planner: all projects + current status
  INDEX-by-type.md                ← planner: deliverables indexed by type
  projects/
    <slug>/
      project-journal.md          ← planner: canonical project state (read this first)
      DECISIONS.md                ← all agents: versioning log (append only)
      synthesis/
        synthesis.md              ← planner: latest artifact synthesis
        versions/                 ← archived prior synthesis versions
      strategy/
        strategy.md               ← planner: latest execution plan
        versions/                 ← archived prior strategy versions
      reviews/
        review-iter<N>.md         ← evaluator: post-delivery retrospective
      output/
        STATUS.md                 ← generator: running status of all steps
        sprint-contract-cp<N>.md  ← evaluator: per-checkpoint acceptance criteria
        eval-verdict-cp<N>-iter<M>.md   ← evaluator: grading record
        eval-feedback-cp<N>-iter<M>.md  ← evaluator: rework instructions
        eval-escalation-cp<N>.md        ← evaluator: escalation report (if cap reached)
        <playbook-defined subfolders>   ← generator: deliverables per playbook layout
```

## Playbook-Defined Output Subfolders

Each playbook declares its own step subfolder layout under `output/`. Examples:

### bi-dashboard playbook

```
output/
  01-requirement/
    01-requirement.md
    versions/
  02-kpi/
    02-kpi-dictionary.md
    VERSION-INDEX.md
    versions/
  03-alignment/
    03-alignment-summary.md
    versions/
  04-discovery/
    04-source-map.md
    versions/
  05-profiling/
    05-data-quality-report.md
    versions/
  06-semantic-model/
    06-semantic-model.md
    versions/
  07-sql-build/
    07-sql-models.md
    versions/
  08-wireframe/
    08-wireframe.md
    versions/
  09-build/
    09-build-spec.md
    versions/
  10-validation/
    10-qa-report.md
    versions/
  11-documentation/
    11-knowledge-pack.md
    versions/
  12-release/
    12-release-checklist.md
    versions/
```

### dbt-data-product playbook

```
output/
  01-requirement/
  02-discovery/
  03-profiling/
  04-semantic-model/
  05-models/
    <model-name>.sql
    versions/
  06-tests/
    schema.yml
    versions/
  07-documentation/
    docs.md
    versions/
  08-release/
```

### analysis-deep-dive playbook

```
output/
  01-requirement/
  02-discovery/
  03-findings/
  04-narrative/
  05-review/
```

## Artifact Library Tree

```
artifacts/
  CATALOG.md                      ← collector: master artifact index
  CHANGELOG.md                    ← collector: artifact evolution log
  .source-registry.md             ← collector: MCP server → source_type map
  ROLLUP-current-state.md         ← collector: current-state rollup (overwritten each run)
  confluence/
    <artifact-id>-<slug>.md       ← topic-based artifact files
  jira/
  teams-chat/
  meeting-transcript/
  ad-hoc/
  versions/                       ← archived superseded artifacts
```

## Slug Conventions

Project slugs are kebab-case, max 40 characters:
- "Sales Dashboard Q2" → `sales-dashboard-q2`
- "Case Management Reporting (Collections)" → `case-mgmt-collections`
- "Customer Churn Analysis" → `customer-churn-analysis`

Reuse the same slug on all invocations for a project. The planner writes the slug to `project-journal.md` on first run; all subsequent agents read and reuse it.

## File Naming Rules

| File type | Pattern | Example |
|---|---|---|
| Step deliverables | `<NN>-<short-name>.md` | `04-source-map.md` |
| Archived versions | `<filename>-v<N>-<YYYYMMDD>.md` | `04-source-map-v1-20260517.md` |
| Sprint contracts | `sprint-contract-cp<N>.md` | `sprint-contract-cp2.md` |
| Eval verdicts | `eval-verdict-cp<N>-iter<M>.md` | `eval-verdict-cp2-iter1.md` |
| Eval feedback | `eval-feedback-cp<N>-iter<M>.md` | `eval-feedback-cp2-iter1.md` |
| Eval escalation | `eval-escalation-cp<N>.md` | `eval-escalation-cp2.md` |
| Retrospective | `review-iter<N>.md` | `review-iter1.md` |
| Artifacts | `ART-<YYYYMMDD>-<NNN>-<slug>.md` | `ART-20260517-001-q1-retention-brief.md` |

## STATUS.md

The generator maintains `docs/projects/<slug>/output/STATUS.md` as a running record:

```markdown
# Delivery Status: <project_name>

**Playbook:** <playbook-name>
**Started:** <date>
**Last updated:** <date>

| Step | Status | Checkpoint | Eval verdict | Iterations |
|---|---|---|---|---|
| 01-requirement | ✅ COMPLETE | CP1 | PASS | 1 |
| 02-kpi | ✅ COMPLETE | CP1 | PASS | 2 |
| 03-alignment | 🔄 IN PROGRESS | CP1 | — | — |
| 04-discovery | ⏳ PENDING | CP2 | — | — |
```

Status values: `⏳ PENDING` | `🔄 IN PROGRESS` | `✅ COMPLETE` | `❌ BLOCKED` | `⏫ ESCALATED`
