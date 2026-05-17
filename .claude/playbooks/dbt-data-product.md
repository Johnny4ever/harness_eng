---
name: dbt-data-product
description: End-to-end dbt data product delivery. From raw stakeholder request through requirements, discovery, semantic modelling, SQL build with tests and documentation, QA, release, and handover. No BI layer — output is a curated mart layer consumed by BI tools, APIs, or downstream teams.
triggers: [dbt, data model, mart, data product, data pipeline, transform, staging model, fact table, dimension, build a model, data engineering]
output_root: docs/projects/<slug>/output/
dbt_target: core       # override per project: core | cloud
max_iterations_default: 5
---

# Playbook: dbt Data Product

## Overview

This playbook delivers a production-ready dbt data product: a set of curated, tested, and documented mart-layer models that serve as a reliable analytical foundation. It covers the full lifecycle from requirements through QA and release, but stops before any BI layer (no wireframes, no dashboard build).

Use `bi-dashboard` instead when the primary deliverable is a dashboard. Use this playbook when the deliverable is the data layer itself — e.g. a new `fct_orders` mart, a customer 360 dimension, or a complete dbt package for a domain.

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
    notes: >
      In this playbook, "KPIs" are the business measures the mart must enable —
      not necessarily dashboard KPIs. Focus on grain, calculation logic, and
      which model columns will serve each measure.

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

  - id: 07b-tests
    skill: dbt/test-design
    output: 07-sql-build/07-dbt-tests.md
    depends_on: [07-sql-build, 06-semantic-model, 05-profiling]

  - id: 07c-docs
    skill: dbt/documentation
    output: 07-sql-build/07-dbt-docs.md
    depends_on: [07-sql-build, 07b-tests, 06-semantic-model, 02-kpi]

  - id: 08-validation
    skill: bi/validation
    output: 08-validation/08-qa-report.md
    depends_on: [07-sql-build, 07b-tests, 07c-docs]
    notes: >
      Validation in this playbook focuses on model-layer correctness:
      row counts, measure reconciliation against source, test pass rates.
      No dashboard UX checks apply.

  - id: 09-documentation
    skill: generic/documentation
    output: 09-documentation/09-knowledge-pack.md
    depends_on: [08-validation]

  - id: 10-release
    skill: generic/release-checklist
    output: 10-release/10-release-checklist.md
    depends_on: [08-validation]
    parallel_with: [09-documentation]
```

## Checkpoints

```yaml
checkpoints:
  - id: cp1
    name: Requirements & Measures
    after_steps: [01-requirement, 02-kpi, 03-alignment]
    human_gate: true
    max_iterations: 5
    eval_contract_hints:
      - "All requested business measures appear in the measure dictionary"
      - "Every measure has grain, calculation logic, and source system identified"
      - "Alignment summary has explicit MVP vs Phase 2 split"
      - "All blocking decisions have an owner and deadline"

  - id: cp2
    name: Data & Model
    after_steps: [04-discovery, 05-profiling, 06-semantic-model, 07-sql-build, 07b-tests, 07c-docs]
    human_gate: true
    max_iterations: 5
    eval_contract_hints:
      - "Every MVP measure has a source mapping or a documented gap reason"
      - "Every fact table has grain declared; every dimension has surrogate key"
      - "Every staging model references only its declared source table"
      - "schema.yml has not_null + unique on every primary key"
      - "Every FK column has a relationships test"
      - "Every gold-tier model has a grain statement in its description"
      - "Source freshness defined for every raw source"
      - "Untested columns section present with justifications"

  - id: cp3
    name: QA & Release
    after_steps: [08-validation, 09-documentation, 10-release]
    human_gate: true
    max_iterations: 3
    eval_contract_hints:
      - "QA report shows overall PASS"
      - "Every measure reconciled against independent reference within tolerance"
      - "All dbt tests pass (zero failing tests)"
      - "Release checklist shows all smoke tests PASS"
      - "Knowledge pack covers model catalogue, data dictionary, and known limitations"
```

## Cross-Cutting Skills

```yaml
cross_cutting:
  - skill: generic/source-enablement
    trigger: discovery output contains source_enablement items
    output: 11-source-enablement/11-enablement-tracker.md

  - skill: generic/governance-check
    trigger: after cp1 (measure dictionary exists) and after cp2 (semantic model exists)
    output: 12-governance/12-governance-report.md
```

## Parallelisation Rules

| Steps | Can run in parallel | Condition |
|---|---|---|
| 02-kpi + 04-discovery | Yes | Both depend only on 01-requirement |
| 07b-tests + 07c-docs | No | 07c-docs reads 07b-tests output |
| 09-documentation + 10-release | Yes | Both depend only on 08-validation |
| All others | No | Sequential dependencies |

## dbt Target Context

The `dbt_target` front-matter variable controls how `dbt/model-build` generates run commands:

| Value | Run command style | Profile |
|---|---|---|
| `core` | `dbt run --select <model>` | Local profiles.yml |
| `cloud` | dbt Cloud job trigger via API | dbt Cloud environment |

## Output Structure

```
docs/projects/<slug>/output/
  STATUS.md
  sprint-contract-cp<N>.md
  eval-verdict-cp<N>-iter<M>.md
  eval-feedback-cp<N>-iter<M>.md
  01-requirement/
    01-requirement.md
  02-kpi/
    02-kpi-dictionary.md
  03-alignment/
    03-alignment-summary.md
  04-discovery/
    04-source-map.md
  05-profiling/
    05-quality-report.md
  06-semantic-model/
    06-semantic-model.md
  07-sql-build/
    07-sql-models.md
    07-dbt-tests.md
    07-dbt-docs.md
  08-validation/
    08-qa-report.md
  09-documentation/
    09-knowledge-pack.md
  10-release/
    10-release-checklist.md
  11-source-enablement/    (if triggered)
  12-governance/           (if triggered)
```
