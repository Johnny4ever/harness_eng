---
name: data/semantic-modeling
description: Design the analytical fact/dimension model, declare grains and relationships, and produce a blueprint for the SQL/dbt build step. Does not write SQL.
inputs:
  - 04-discovery/04-source-map.md
  - 05-profiling/05-quality-report.md
  - 02-kpi/02-kpi-dictionary.md
outputs:
  - 06-semantic-model/06-semantic-model.md
model_tier_hint: sonnet
used_by_playbooks: [bi-dashboard, dbt-data-product]
intent_tags: [semantic model, dimensional model, fact table, dimension table, grain, star schema, data model]
---

# Skill: Semantic Model Design

## Boundaries

Does NOT: write SQL or dbt code (that is `dbt/model-build`), confirm source feasibility (that is `data/discovery`), profile data quality (that is `data/quality-profiling`). Produces a blueprint only.

## Procedure

### Step 1 — Establish the central grain
Identify the grain for each fact table:
- What does one row represent? (e.g. "one subscription charge per customer per month")
- Which KPIs are supported at this grain?
- Is this an additive, semi-additive, or non-additive fact?

If KPIs require different grains, design separate fact tables. Document the reason.

### Step 2 — Design fact tables
For each fact table:
- **Name** (snake_case, prefixed `fct_`)
- **Grain declaration** — one sentence
- **Measures** — columns that are aggregated (SUM, COUNT, AVG)
- **Degenerate dimensions** — descriptive columns stored on the fact (e.g. order_status)
- **Foreign keys** — which dimension tables it joins to
- **Source tables** — which raw/staging tables feed it (from source map)
- **Transformation notes** — any dedup, null filter, or business logic from quality report

### Step 3 — Design dimension tables
For each dimension:
- **Name** (prefixed `dim_`)
- **Surrogate key** — system-generated integer key
- **Natural key** — business identifier from source
- **Key attributes** — descriptive columns the fact joins to
- **Slowly changing** — Type 1 (overwrite) or Type 2 (history) — state which and why
- **Source table**

### Step 4 — Define relationships
Draw the star/snowflake schema as a text diagram:

```
fct_orders ──── dim_customer
     │
     ├────────── dim_product
     │
     └────────── dim_date
```

For each relationship: state the join key and cardinality (many-to-one, one-to-one).

### Step 5 — Map KPIs to the model
For every KPI in the dictionary, show how it is calculated from the model:

| KPI | Formula using model columns | Grain | Filter |
|---|---|---|---|
| `churn_rate` | `COUNT(fct_sub WHERE status='cancelled') / COUNT(fct_sub at period start)` | monthly | paying customers only |

### Step 6 — Write the semantic model

```markdown
---
step: 06
slug: <project-slug>
version: v1
created: <date>
fact_tables: <N>
dimension_tables: <N>
---

# Semantic Model: <Project Name>

## Schema Diagram
<text diagram>

## Fact Tables
### fct_<name>
**Grain:** <one row per X>
**Measures:** <list>
**Foreign keys:** <list>
**Sources:** <list>
**Transformation notes:** <from quality report>

## Dimension Tables
### dim_<name>
**Surrogate key:** <col>
**Natural key:** <col>
**Key attributes:** <list>
**SCD type:** Type 1 / Type 2
**Source:** <table>

## KPI-to-Model Mapping
| KPI | Formula | Grain | Filter |
|---|---|---|---|

## Design Decisions
| Decision | Options considered | Choice | Reason |
|---|---|---|---|
```

## Self-Check Checklist
- [ ] Every fact table has grain declared in one sentence
- [ ] Every dimension table has surrogate key and natural key declared
- [ ] Every KPI from the MVP scope appears in the KPI-to-model mapping
- [ ] Schema diagram shows all join relationships
- [ ] Every transformation note from the quality report is referenced in the relevant fact table
- [ ] SCD type declared for every dimension
