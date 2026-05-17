---
name: bi/kpi-definition
description: Convert business language from a requirement document into precise, reusable KPI definitions with calculation logic, grain, filters, and ownership.
inputs:
  - requirement document (from prior step or user-provided context)
  - business glossaries or existing metric dictionaries (optional)
  - legacy dashboard definitions (optional)
outputs:
  - 02-kpi/02-kpi-dictionary.md
model_tier_hint: sonnet
used_by_playbooks: [bi-dashboard, kpi-proof]
intent_tags: [kpi, metric, definition, kpi dictionary, business metric, measure]
---

# Skill: KPI Definition

## When to Use

Load this skill when the generator needs to convert business requirements into a structured KPI dictionary. Always runs after a requirement document exists (step 01 or user-provided context). Always runs before data discovery (step 04) — discovery needs confirmed KPI names to map.

## Boundaries

This skill does **not**:
- Confirm source feasibility or map KPIs to specific tables/columns — that is `data/discovery`
- Prioritise or scope the MVP — that is `generic/stakeholder-alignment`
- Write SQL or dbt models — that is `dbt/model-build`
- Design dashboard visuals — that is `bi/wireframe-ux`
- Assess data quality — that is `data/quality-profiling`

Candidate source systems may be proposed at a high level but must be labelled **"unconfirmed — pending discovery"**.

## Inputs

| Input | Source | Notes |
|---|---|---|
| Requirement document | Prior step output or user context | Must exist before this skill runs |
| Business glossary | Artifact library or user-provided | Optional — note if absent |
| Legacy KPI definitions | Artifact library or user-provided | Optional — flag conflicts if present |

## Procedure

### Step 1 — Scan for candidate metrics

Read the requirement document and identify every metric by looking for:
- Explicit KPI lists ("we want to track X, Y, Z")
- Descriptions of success ("we need to know if conversion improves")
- Goal-oriented language ("we care about", "key outcomes", "targets")
- Implicit metrics ("how many customers churned" → `churn_count` + `churn_rate`)

List all candidates before defining any of them.

### Step 2 — Normalise and de-duplicate

- Consolidate synonyms into one canonical name (`monthly active users` = `MAU` = `active_users_monthly` → canonical: `monthly_active_users`)
- Record aliases in the `aliases` field of each KPI
- Flag any KPI that appears to overlap with an existing enterprise definition

### Step 3 — Define each KPI completely

For every KPI, populate all fields in the KPI record (see template below). Do not leave fields blank — if unknown, write "TBD — open question #N" and record the open question in the ambiguity log.

**KPI record template:**

```markdown
### <Metric Name>

| Field | Value |
|---|---|
| Canonical name | `snake_case_name` |
| Aliases | comma-separated list |
| Business description | Plain English: what does this number tell a business user? |
| Calculation | Numerator / Denominator (or formula). Conceptual, not SQL. |
| Metric type | snapshot \| event-based \| cumulative \| rate/ratio |
| Grain | The level of detail: e.g. `user_id × date`, `order_id`, `monthly` |
| Date handling | As-of date or period? Rolling window or fixed? e.g. "last 30 calendar days" |
| Filters / exclusions | What is excluded: test accounts, internal users, refunded orders, etc. |
| Source candidates | System or table names — label as **unconfirmed** |
| Business owner | Team or person accountable for this metric |
| Validation method | How to check if the number is right: spot check, reconcile to report X, etc. |
```

### Step 4 — Classify metric type

Apply these definitions consistently:

| Type | Definition | Example |
|---|---|---|
| Snapshot | Value at a point in time | `active_subscriptions` as of today |
| Event-based | Counts or sums of discrete events | `orders_placed` in a period |
| Cumulative | Running total that never decreases | `total_revenue_since_launch` |
| Rate/ratio | Numerator ÷ denominator | `conversion_rate` = signups / visits |

### Step 5 — Handle date logic explicitly

For every KPI state:
- **Period type**: calendar day / week / month / quarter / year — or rolling window (last N days)?
- **Snapshot date**: if snapshot, which date is used (today, end of period, as-of date)?
- **Attribution**: if event-based, which event timestamp drives the date (order placed, order completed, payment received)?

Never leave date logic implicit. Ambiguous date logic is the leading cause of metric reconciliation failures in QA.

### Step 6 — Write the ambiguity log

Create a section at the end of the dictionary for:
- **Conflicts**: cases where two business units define the same metric differently
- **Open questions**: information needed that is not in the requirement doc (number them OQ-1, OQ-2, ...)
- **Assumptions**: decisions made in the absence of clarity (must be validated with stakeholders)

### Step 7 — Write the output file

Write to: `docs/projects/<slug>/output/02-kpi/02-kpi-dictionary.md`

Apply the versioning protocol (`rules/versioning-protocol.md`) if a prior version exists.

**Output file structure:**

```markdown
---
step: 02
slug: <project-slug>
version: v1
created: <date>
kpi_count: <N>
open_questions: <N>
---

# KPI Dictionary: <Project Name>

## Summary
<N> KPIs defined. <N> open questions. <N> conflicts flagged.

## KPI Records
[one section per KPI using the template from Step 3]

## Ambiguity Log

### Conflicts
| ID | KPI | Conflict description | Resolution |
|---|---|---|---|

### Open Questions
| ID | KPI | Question | Owner | Due |
|---|---|---|---|---|

### Assumptions
| ID | KPI | Assumption | Risk if wrong |
|---|---|---|---|
```

## Self-Check Checklist

Run through this before writing the handoff block:

- [ ] Every KPI from the requirement doc is present (none skipped)
- [ ] Every KPI record has all 11 fields populated (no silent blanks)
- [ ] Every "TBD" has a corresponding open question number in the ambiguity log
- [ ] Date logic is explicit for every KPI (no "as needed" or "standard period")
- [ ] Source candidates are labelled "unconfirmed" (none presented as confirmed)
- [ ] Metric type is assigned for every KPI using the 4-type classification
- [ ] Ambiguity log exists and is non-empty (there are always open questions on first draft)
- [ ] Output file is written to the correct path
- [ ] Versioning protocol applied if prior version existed
