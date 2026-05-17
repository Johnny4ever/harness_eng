---
name: bi-dashboard
description: End-to-end BI dashboard delivery. From raw stakeholder request through requirements, discovery, modelling, SQL, wireframe, build, QA, release, and documentation.
triggers: [dashboard, report, BI, visualisation, analytics, KPI dashboard, build a dashboard, sales dashboard, finance report]
output_root: docs/projects/<slug>/output/
bi_platform: powerbi   # override per project: powerbi | tableau | looker
dbt_target: core       # override per project: core | cloud
max_iterations_default: 5
---

# Playbook: BI Dashboard

## Steps

```yaml
steps:
  - id: 01-requirement
    skill: generic/requirement-intake
    output: 01-requirement/01-requirement.md
    depends_on: []

  - id: 02-kpi
    skill: bi/kpi-definition
    output: 02-kpi/02-kpi-dictionary.md
    depends_on: [01-requirement]
    parallel_with: [04-discovery]

  - id: 03-alignment
    skill: generic/stakeholder-alignment
    output: 03-alignment/03-alignment-summary.md
    depends_on: [01-requirement, 02-kpi]

  - id: 04-discovery
    skill: data/discovery
    output: 04-discovery/04-source-map.md
    depends_on: [01-requirement, 02-kpi]
    parallel_with: [02-kpi]

  - id: 05-profiling
    skill: data/quality-profiling
    output: 05-profiling/05-quality-report.md
    depends_on: [04-discovery]

  - id: 06-semantic-model
    skill: data/semantic-modeling
    output: 06-semantic-model/06-semantic-model.md
    depends_on: [04-discovery, 05-profiling, 02-kpi]

  - id: 07-sql-build
    skill: dbt/model-build
    output: 07-sql-build/07-sql-models.md
    depends_on: [06-semantic-model, 05-profiling]

  - id: 08-wireframe
    skill: bi/wireframe-ux
    output: 08-wireframe/08-wireframe.md
    depends_on: [04-discovery, 02-kpi, 03-alignment]

  - id: 09-build
    skill: bi/dashboard-build
    output: 09-build/09-build-spec.md
    depends_on: [08-wireframe, 06-semantic-model, 02-kpi]

  - id: 10-validation
    skill: bi/validation
    output: 10-validation/10-qa-report.md
    depends_on: [09-build]

  - id: 11-documentation
    skill: generic/documentation
    output: 11-documentation/11-knowledge-pack.md
    depends_on: [10-validation]

  - id: 12-release
    skill: generic/release-checklist
    output: 12-release/12-release-checklist.md
    depends_on: [10-validation]
    parallel_with: [11-documentation]
```

## Checkpoints

```yaml
checkpoints:
  - id: cp1
    name: Requirements & KPI Definition
    after_steps: [01-requirement, 02-kpi, 03-alignment]
    human_gate: true
    max_iterations: 5
    eval_contract_hints:
      - "All requested KPIs named in requirement doc appear in the KPI dictionary"
      - "Every KPI has all 11 fields populated with real content (no TBD-only fields)"
      - "Alignment summary has explicit MVP vs Phase 2 split"
      - "All blocking decisions have an owner and deadline"

  - id: cp2
    name: Data & Model
    after_steps: [04-discovery, 05-profiling, 06-semantic-model, 07-sql-build]
    human_gate: true
    max_iterations: 5
    eval_contract_hints:
      - "Every MVP KPI has a source mapping or a documented gap reason"
      - "Every feasible KPI has grain and join path documented"
      - "Every fact table has grain declared; every dimension has surrogate key"
      - "Every KPI maps to a model column in the KPI-to-model mapping"
      - "Every staging model references only its declared source table"
      - "schema.yml has not_null + unique on every primary key"

  - id: cp3
    name: Design
    after_steps: [08-wireframe]
    human_gate: true
    max_iterations: 5
    eval_contract_hints:
      - "Every MVP KPI appears on at least one dashboard page"
      - "Every visual references only columns confirmed in the source map"
      - "Sample values present for every chart"
      - "Filter scope (global vs local) specified for every filter"

  - id: cp4
    name: Build & Release
    after_steps: [09-build, 10-validation, 11-documentation, 12-release]
    human_gate: true
    max_iterations: 3
    eval_contract_hints:
      - "QA report shows overall PASS"
      - "Every KPI reconciled against an independent reference within tolerance"
      - "Release checklist shows all smoke tests PASS"
      - "Knowledge pack covers all 5 sections with no missing KPIs"
```

## Cross-Cutting Skills

Triggered by the generator at any checkpoint when conditions are met:

```yaml
cross_cutting:
  - skill: generic/source-enablement
    trigger: discovery output contains source_enablement items
    output: 13-source-enablement/13-enablement-tracker.md

  - skill: generic/governance-check
    trigger: after cp1 (KPI dictionary exists) and after cp2 (semantic model exists)
    output: 14-governance/14-governance-report.md
```

## Parallelisation Rules

| Steps | Can run in parallel | Condition |
|---|---|---|
| 02-kpi + 04-discovery | Yes | Both depend only on 01-requirement |
| 11-documentation + 12-release | Yes | Both depend only on 10-validation |
| All others | No | Sequential dependencies |

## Output Structure

```
docs/projects/<slug>/output/
  STATUS.md
  sprint-contract-cp<N>.md
  eval-verdict-cp<N>-iter<M>.md
  eval-feedback-cp<N>-iter<M>.md
  01-requirement/
  02-kpi/
  03-alignment/
  04-discovery/
  05-profiling/
  06-semantic-model/
  07-sql-build/
  08-wireframe/
  09-build/
  10-validation/
  11-documentation/
  12-release/
  13-source-enablement/    (if triggered)
  14-governance/           (if triggered)
```
