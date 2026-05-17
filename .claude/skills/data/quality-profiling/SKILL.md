---
name: data/quality-profiling
description: Profile candidate sources identified in discovery for completeness, accuracy, consistency, timeliness, uniqueness, and validity. Assess fitness for KPI use and surface remediation options.
inputs:
  - 04-discovery/04-source-map.md
  - direct database access via MCP (preferred) or sample data provided by user
outputs:
  - 05-profiling/05-quality-report.md
model_tier_hint: sonnet
used_by_playbooks: [bi-dashboard, dbt-data-product]
intent_tags: [data quality, profiling, completeness, null rate, freshness, accuracy, fitness]
---

# Skill: Data Quality Profiling

## Boundaries

Does NOT: fix data quality issues (that is upstream source team work), design the semantic model (that is `data/semantic-modeling`), write production SQL. Assesses fitness — does not remediate.

Uses Snowflake MCP or other database MCP when available for direct profiling queries. Falls back to user-provided samples or documentation when MCPs are unavailable — note the limitation in the report.

## Six Quality Dimensions

Profile each source table across all six dimensions:

| Dimension | What to measure |
|---|---|
| **Completeness** | Null rate per key field. What % of rows are usable? |
| **Accuracy** | Do values match expected ranges / known totals? Spot-check against a known reference. |
| **Consistency** | Same entity represented the same way across tables? (e.g. customer_id format) |
| **Timeliness** | How fresh is the latest record? Does max(updated_at) meet the refresh requirement? |
| **Uniqueness** | Are primary/join keys actually unique? Duplicate rate on key fields. |
| **Validity** | Do values conform to expected formats, enums, or ranges? |

## Procedure

### Step 1 — Prioritise tables to profile
From the source map, profile only the tables marked ✅ or ⚠️ feasible. Do not spend time profiling gap KPI sources.

For each table, run (or request) the following profile queries:
- Row count
- Null rate on every key field (metric numerator, denominator, date, join key, filter fields)
- Duplicate rate on the declared primary key
- Min/max of the date field (history depth verification)
- Max(updated_at) or equivalent (freshness check)
- Value distribution for critical enum/category fields (top 10 values)

### Step 2 — Score each table

| Score | Meaning |
|---|---|
| ✅ Fit | All dimensions acceptable. No remediation needed. |
| ⚠️ Conditional | One dimension has an issue that can be handled in the transformation layer (e.g. null filter, dedup logic) |
| ❌ Not fit | A dimension issue is blocking and cannot be handled downstream |

### Step 3 — Document remediation options
For every ⚠️ table: specify the exact transformation-layer fix (e.g. "filter WHERE status IS NOT NULL", "dedup using ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY updated_at DESC)"). This feeds directly into `dbt/model-build`.

### Step 4 — Write the quality report

```markdown
---
step: 05
slug: <project-slug>
version: v1
created: <date>
profiled_tables: <N>
fit_tables: <N>
conditional_tables: <N>
not_fit_tables: <N>
---

# Data Quality Report: <Project Name>

## Executive Summary
<2-3 sentences: overall data fitness, key risks, recommended actions>

## Table Profiles

### <schema.table_name>
**KPIs supported:** <list>
**Row count:** <N>
**Profile method:** direct query / user-provided sample / documentation only

| Dimension | Finding | Score |
|---|---|---|
| Completeness | <field>: X% null | ✅ / ⚠️ / ❌ |
| Accuracy | reconciled to <reference>: variance X% | |
| Consistency | customer_id format consistent across joins | |
| Timeliness | max(updated_at) = <date>, lag = <N hours> | |
| Uniqueness | <key>: X duplicates found | |
| Validity | status field: X% unexpected values | |

**Overall:** ✅ / ⚠️ / ❌
**Remediation:** <if ⚠️: exact fix to apply in transformation>

## Blocking Issues
| Table | Dimension | Issue | Impact on KPI | Options |
|---|---|---|---|---|

## Recommended Transformation Fixes
| Table | Fix | Implemented in step |
|---|---|---|
```

## Self-Check Checklist
- [ ] All feasible/conditional tables from source map are profiled
- [ ] All 6 dimensions assessed for each table (even if "not checked — no MCP access")
- [ ] Every ⚠️ table has a specific remediation instruction
- [ ] Every ❌ table has a stated impact on the KPI it supports
- [ ] Freshness checked against the dashboard's required refresh frequency
