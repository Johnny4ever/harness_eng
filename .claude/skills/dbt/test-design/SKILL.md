---
name: dbt/test-design
description: Write dbt test suites for staging and mart models — schema.yml not_null/unique/accepted_values/relationships tests, custom singular tests for business rules, and a test coverage report.
inputs:
  - 07-sql-build/07-sql-models.md (model definitions)
  - 06-semantic-model/06-semantic-model.md (grain and key declarations)
  - 05-profiling/05-quality-report.md (known data quality issues to guard against)
outputs:
  - 07-sql-build/07-dbt-tests.md (schema.yml test blocks + singular test SQL)
model_tier_hint: sonnet
used_by_playbooks: [dbt-data-product]
intent_tags: [dbt, test, testing, schema.yml, data quality, test coverage, singular test, generic test]
---

# Skill: dbt Test Design

## Boundaries

Does NOT: build transformation models (that is `dbt/model-build`), profile raw sources (that is `data/quality-profiling`), or run dbt commands. Produces test YAML and SQL that is appended to the sql-models output file.

## dbt Test Taxonomy

| Test type | Where defined | When to use |
|---|---|---|
| `not_null` | schema.yml | Every column that must never be NULL per semantic model |
| `unique` | schema.yml | Every primary key and natural key |
| `accepted_values` | schema.yml | Status fields, type enumerations, categorical columns |
| `relationships` | schema.yml | Every foreign key to a dimension |
| Singular test (SQL) | tests/ folder | Business rules too complex for generic tests |
| Source freshness | sources.yml | Every raw source table used in staging models |

## Procedure

### Step 1 — Identify all models requiring tests
From `07-sql-models.md`, list every staging model (`stg_*`), intermediate model (`int_*`), and mart model (`fct_*`, `dim_*`).

### Step 2 — Apply mandatory tests per model type

**Every staging model must have:**
- `not_null` + `unique` on the declared primary key
- `not_null` on every foreign key column
- `accepted_values` on every status/type column if the semantic model defines the allowed set
- `relationships` test for every FK that references a dimension

**Every fact table must have:**
- `not_null` + `unique` on the surrogate key (if present) or grain columns
- `not_null` on every measure column that feeds a KPI
- `relationships` test to every dimension it joins

**Every dimension must have:**
- `not_null` + `unique` on the surrogate key
- `not_null` + `unique` on the natural/business key

### Step 3 — Identify custom singular tests
Review `05-quality-report.md` for known data quality risks. For each risk that a generic test cannot catch, write a singular test SQL file.

Common singular test patterns:

```sql
-- test: assert_no_negative_revenue.sql
-- Fails if any row has revenue < 0
select order_id
from {{ ref('fct_orders') }}
where revenue < 0
```

```sql
-- test: assert_trial_end_after_start.sql
-- Fails if trial_end_date precedes trial_start_date
select subscription_id
from {{ ref('fct_subscriptions') }}
where trial_end_date < trial_start_date
```

A singular test **fails** when it returns any rows — write the SELECT to return violating rows only.

### Step 4 — Add source freshness tests
For every `{{ source() }}` reference in staging models, add a `freshness` block to `sources.yml`:

```yaml
sources:
  - name: raw_salesforce
    freshness:
      warn_after: {count: 24, period: hour}
      error_after: {count: 48, period: hour}
    loaded_at_field: _etl_loaded_at
    tables:
      - name: opportunities
```

### Step 5 — Write the test output document

Structure `07-dbt-tests.md` as:

```markdown
---
step: 07-tests
slug: <project-slug>
version: v1
created: <date>
---

# dbt Test Suite: <Project Name>

## Test Coverage Summary
| Model | Primary key tests | FK/relationship tests | Business rule tests | Source freshness |
|---|---|---|---|---|
| stg_salesforce__opportunities | ✅ | ✅ | — | ✅ |
| fct_revenue | ✅ | ✅ | 2 singular tests | — |

## schema.yml Test Blocks

### stg_salesforce__opportunities
\`\`\`yaml
models:
  - name: stg_salesforce__opportunities
    columns:
      - name: opportunity_id
        tests:
          - not_null
          - unique
      - name: account_id
        tests:
          - not_null
          - relationships:
              to: ref('dim_account')
              field: account_id
      - name: stage
        tests:
          - accepted_values:
              values: ['Prospecting', 'Qualification', 'Closed Won', 'Closed Lost']
\`\`\`

## Singular Tests

### tests/assert_no_negative_revenue.sql
\`\`\`sql
select order_id
from {{ ref('fct_revenue') }}
where revenue_amount < 0
\`\`\`

## Source Freshness (sources.yml additions)
\`\`\`yaml
<freshness blocks>
\`\`\`

## Untested Columns
| Model | Column | Reason not tested |
|---|---|---|
```

### Step 6 — Identify gaps
List any column that cannot be tested because:
- Source data has known nulls that are intentional (note why)
- Accepted values set is not defined (flag for stakeholder clarification)
- Business rule requires joining multiple sources (note as future test)

## Self-Check Checklist
- [ ] Every `fct_*` and `dim_*` primary key has `not_null` + `unique`
- [ ] Every FK column has a `relationships` test
- [ ] Every status/type column with a defined value set has `accepted_values`
- [ ] Source freshness defined for every raw source
- [ ] Singular tests return violating rows only (never `count(*) = 0` style)
- [ ] Untested columns section present with reasons
- [ ] Test coverage summary table populated
