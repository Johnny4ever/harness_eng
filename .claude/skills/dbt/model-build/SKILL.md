---
name: dbt/model-build
description: Implement curated data models and transformation logic as SQL/dbt models that realise the approved semantic design. Enforces dbt conventions, layered architecture, and tests.
inputs:
  - 06-semantic-model/06-semantic-model.md
  - 05-profiling/05-quality-report.md (for transformation fixes)
  - existing dbt project structure (if present)
outputs:
  - 07-sql-build/07-sql-models.md (model catalogue + SQL)
model_tier_hint: sonnet
used_by_playbooks: [bi-dashboard, dbt-data-product]
intent_tags: [sql, dbt, transformation, model, build, cte, staging, mart]
---

# Skill: dbt Model Build

## Boundaries

Does NOT: design the semantic model (that is `data/semantic-modeling`), assess data quality (that is `data/quality-profiling`), build dashboards (that is `bi/dashboard-build`). Implements what the semantic model specifies.

Supports both **dbt Core** and **dbt Cloud**. The playbook passes `dbt_target: core | cloud` as context. Cloud-specific features (dbt Explorer, deferral, CI/CD integrations) are only used when `dbt_target: cloud`.

## Layered Architecture

Always implement models in three layers:

```
sources (raw)
  └─ staging (stg_*)    — 1:1 with source, rename + cast + basic cleaning
       └─ intermediate (int_*)  — joins and business logic (optional layer)
            └─ marts (fct_* / dim_*)  — final analytics-ready tables
```

Never reference raw source tables directly from mart models. Always go through staging.

## Procedure

### Step 1 — Scaffold staging models
For each source table in the semantic model:
- Create `stg_<source>__<table>.sql`
- Rename columns to snake_case
- Cast data types explicitly
- Apply null filters and dedup logic from the quality report
- Add `{{ config(materialized='view') }}`

### Step 2 — Build intermediate models (if needed)
When two or more staging models need to be joined before the mart:
- Create `int_<description>.sql`
- One join per intermediate model — avoid multi-join intermediates
- Add `{{ config(materialized='ephemeral') }}` for simple joins, `view` for reused intermediates

### Step 3 — Build fact and dimension models
For each `fct_*` and `dim_*` in the semantic model:
- Implement the KPI-to-model mapping formulas exactly as defined
- Use CTEs — one CTE per logical step (source → rename → filter → aggregate → final)
- For dimensions: generate surrogate key using `{{ dbt_utils.generate_surrogate_key([...]) }}`
- For SCD Type 2 dimensions: use snapshot configuration
- Add `{{ config(materialized='table') }}` for facts and dimensions

### Step 4 — Write schema.yml tests
For every model, add to `schema.yml`:
- `not_null` on all foreign keys and primary keys
- `unique` on all primary keys
- `accepted_values` on enum/status columns
- `relationships` test on every foreign key join

### Step 5 — Validate compilation
If dbt MCP or CLI is available: run `dbt compile` and confirm zero errors. If not available: note in the output that compilation was not verified and flag for manual check.

### Step 6 — Write the model catalogue

```markdown
---
step: 07
slug: <project-slug>
version: v1
created: <date>
models_built: <N>
dbt_target: core | cloud
compiled: true | false | not-verified
---

# SQL / dbt Models: <Project Name>

## Model Inventory
| Model | Layer | Materialisation | Source(s) | KPIs supported |
|---|---|---|---|---|

## Model Code

### stg_<source>__<table>.sql
<SQL>

### fct_<name>.sql
<SQL>

### dim_<name>.sql
<SQL>

## schema.yml
<YAML tests>

## Known Issues / Manual Steps
<anything that needs human review — e.g. unverified compilation, MCP unavailable>
```

## Self-Check Checklist
- [ ] Every source table has a staging model
- [ ] No mart model references raw sources directly
- [ ] Every mart model uses CTEs with one logical step per CTE
- [ ] Surrogate keys generated for all dimension tables
- [ ] schema.yml has not_null + unique on every primary key
- [ ] schema.yml has relationships test on every foreign key
- [ ] dbt_target context applied (Cloud features only if cloud)
- [ ] Compilation status noted (verified / not-verified)
