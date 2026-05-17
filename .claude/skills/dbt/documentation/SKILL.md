---
name: dbt/documentation
description: Write dbt model documentation — description fields in schema.yml, column-level lineage notes, a model catalogue markdown doc, and a data dictionary for downstream consumers.
inputs:
  - 07-sql-build/07-sql-models.md (model SQL and CTE structure)
  - 07-sql-build/07-dbt-tests.md (test coverage, if available)
  - 06-semantic-model/06-semantic-model.md (business definitions and grain)
  - 02-kpi/02-kpi-dictionary.md (KPI definitions and ownership)
outputs:
  - 07-sql-build/07-dbt-docs.md (schema.yml description blocks + model catalogue + data dictionary)
model_tier_hint: sonnet
used_by_playbooks: [dbt-data-product]
intent_tags: [dbt, documentation, data dictionary, model catalogue, schema.yml, lineage, column description]
---

# Skill: dbt Documentation

## Boundaries

Does NOT: build models or write tests (those are `dbt/model-build` and `dbt/test-design`). Writes documentation that lives in schema.yml and as a standalone markdown reference.

Two audiences:
1. **dbt/analytics engineers**: schema.yml descriptions used by `dbt docs generate`
2. **Business users / downstream consumers**: plain-language data dictionary

## Procedure

### Step 1 — Write schema.yml description blocks

For every model, produce a `description` field at model level and for every exposed column.

**Model-level description template:**
```yaml
models:
  - name: fct_revenue
    description: >
      One row per recognised revenue transaction. Grain: one row per invoice_id.
      Joins to dim_customer (customer dimension) and dim_product (product dimension).
      Primary source: raw_stripe.charges joined with raw_salesforce.opportunities.
      Rebuilt daily at 06:00 UTC.
    meta:
      owner: data-team
      domain: finance
      tier: gold   # bronze | silver | gold
```

**Column-level description template:**
```yaml
    columns:
      - name: revenue_amount_usd
        description: >
          Recognised revenue in USD at the time of invoice. Excludes refunds (see
          refund_amount_usd). Currency conversion uses the exchange rate at invoice
          date from dim_fx_rates.
        meta:
          kpi: monthly_recurring_revenue
          pii: false
```

Rules:
- Model description must state **grain** explicitly ("One row per X")
- Column descriptions must state what null means (if nullable) or confirm not-null
- For measures: describe what is included/excluded and any currency/unit
- For foreign keys: name the dimension they join to
- `meta.kpi` links the column to a KPI from the dictionary (use canonical KPI name)

### Step 2 — Assign model tiers

| Tier | Layer | Consumers |
|---|---|---|
| bronze | staging | Internal analytics engineering only |
| silver | intermediate | Internal analytics engineering only |
| gold | mart | BI tools, business users, APIs |

Only `gold` tier models are included in the consumer-facing data dictionary.

### Step 3 — Write the Model Catalogue

Produces a reference table for analytics engineers:

```markdown
## Model Catalogue

| Model | Tier | Grain | Primary key | Sources | Owner | Refresh |
|---|---|---|---|---|---|---|
| stg_stripe__charges | bronze | One row per charge_id | charge_id | raw_stripe.charges | data-team | daily 04:00 UTC |
| fct_revenue | gold | One row per invoice_id | invoice_surrogate_key | stg_stripe__charges, stg_salesforce__opportunities | data-team | daily 06:00 UTC |

## Lineage Summary
<short prose: source → staging → intermediate → mart flow; which systems feed which facts>
```

### Step 4 — Write the Data Dictionary

Consumer-facing plain-language reference. Include only `gold` tier models.

```markdown
## Data Dictionary

### fct_revenue — Revenue Transactions

> One transaction per recognised invoice. Rebuilt daily. Use this table for all revenue reporting.

| Column | Type | Description | Example |
|---|---|---|---|
| invoice_id | string | Unique identifier for the invoice (from Stripe) | ch_3NxK... |
| customer_id | string | Foreign key to dim_customer | CUST-00142 |
| revenue_amount_usd | decimal | Recognised revenue in USD, excluding refunds | 1250.00 |
| invoice_date | date | Date invoice was raised (not payment date) | 2026-05-01 |
| product_tier | string | Product tier at time of invoice: Starter / Pro / Enterprise | Pro |

**KPIs calculated from this table:**
- Monthly Recurring Revenue (MRR)
- Annual Recurring Revenue (ARR)

**Known limitations:**
- Multi-currency invoices converted at invoice date; historical rates may differ from actuals
- Refunds are recorded separately in fct_refunds; this table shows gross revenue only
```

### Step 5 — Produce the output document

Structure `07-dbt-docs.md` as:

```markdown
---
step: 07-docs
slug: <project-slug>
version: v1
created: <date>
---

# dbt Documentation: <Project Name>

## schema.yml Description Blocks
<full YAML blocks for all models — copy-paste ready>

## Model Catalogue
<table + lineage summary>

## Data Dictionary
<gold-tier models only, plain language>

## Documentation Gaps
| Model | Column | Gap | Recommended action |
|---|---|---|---|
```

### Step 6 — Flag documentation gaps
Columns where descriptions could not be completed because business meaning is unclear:
- Note the column name and model
- State what information is needed (e.g. "what does status = NULL mean?")
- Flag to stakeholder-alignment if the gap is blocking

## Self-Check Checklist
- [ ] Every model has a grain statement in its description
- [ ] Every column has a non-trivial description (not just "the X column")
- [ ] `meta.tier` set on every model (bronze/silver/gold)
- [ ] Only gold-tier models appear in the Data Dictionary
- [ ] KPI links (`meta.kpi`) populated for measure columns that drive KPIs
- [ ] Known limitations section present in data dictionary for each table
- [ ] Documentation gaps table present (even if empty)
