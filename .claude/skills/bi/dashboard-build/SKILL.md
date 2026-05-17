---
name: bi/dashboard-build
description: Implement the approved wireframe as a working dashboard in the target BI platform (Power BI, Tableau, Looker). Produces build specifications and BI-layer calculation code.
inputs:
  - 08-wireframe/08-wireframe.md
  - 06-semantic-model/06-semantic-model.md
  - 02-kpi/02-kpi-dictionary.md
outputs:
  - 09-build/09-build-spec.md
model_tier_hint: sonnet
used_by_playbooks: [bi-dashboard]
intent_tags: [dashboard build, power bi, tableau, looker, dax, lkml, calculated field, publish]
---

# Skill: Dashboard Build

## Boundaries

Does NOT: redesign the wireframe (consumes it as approved spec), redefine KPIs, write dbt/SQL models (that is `dbt/model-build`). Implements exactly what the wireframe specifies at the BI layer.

BI-layer calculations (DAX, Tableau calculated fields, LookML measures) are written here — not production SQL. Heavy transformations belong in `dbt/model-build`, not the BI layer.

## Platform-Specific Guidance

The playbook passes `bi_platform: powerbi | tableau | looker` as context.

| Platform | Calculation language | Connection type | Build artefact |
|---|---|---|---|
| Power BI | DAX | DirectQuery / Import | .pbix spec + DAX measures |
| Tableau | Tableau calculated fields | Live / Extract | .twbx spec + calcs |
| Looker | LookML | Live (always) | view.lkml + explore.lkml |

## Procedure

### Step 1 — Map wireframe pages to platform constructs
Translate each wireframe page into platform-specific terms:
- Power BI: report page + visuals + slicers
- Tableau: worksheet + dashboard + filters
- Looker: explore + dashboard tiles + filters

### Step 2 — Write BI-layer calculations
For each KPI that requires a calculated measure (not a simple column aggregate):
- Write the formula in the platform's language
- Reference the curated model columns (from semantic model), not raw source columns
- Label each calculation with its KPI canonical name

```
-- Example Power BI DAX
Churn Rate =
DIVIDE(
    CALCULATE(COUNTROWS(fct_subscriptions), fct_subscriptions[status] = "cancelled"),
    CALCULATE(COUNTROWS(fct_subscriptions), DATEADD(fct_subscriptions[date], -1, MONTH))
)
```

### Step 3 — Specify each visual
For every visual in the wireframe, produce a build specification:

```markdown
#### Visual: MRR Trend (Line Chart)
- Chart type: Line
- X-axis: dim_date[month_start] — monthly
- Y-axis: [MRR measure] — format: $#,##0
- Filters applied: date range slicer (global)
- Tooltip: month label + MRR value + MoM change %
- Drill-through: none
```

### Step 4 — Specify data connections
- Connection type (DirectQuery / Import / Live)
- Which curated models are connected (`fct_*`, `dim_*`)
- Refresh schedule configured (matches requirement doc)
- Row-level security rules (if applicable)

### Step 5 — Write the build specification

```markdown
---
step: 09
slug: <project-slug>
version: v1
created: <date>
platform: <powerbi|tableau|looker>
pages: <N>
calculated_measures: <N>
---

# Build Specification: <Project Name>

## Platform & Connection
Platform: <name>
Connection type: <DirectQuery|Import|Live>
Curated models connected: <list of fct_* and dim_* tables>
Refresh schedule: <per requirement>

## BI-Layer Calculations
### <KPI Canonical Name>
<formula in platform language>

## Visual Specifications
### Page: <Name>
#### Visual: <Name>
<spec per visual>

## Row-Level Security
<rules if applicable, or "not required">

## Manual Steps Required
<anything that cannot be fully specified in Markdown — e.g. actual .pbix file creation>
```

## Self-Check Checklist
- [ ] Every wireframe page has a corresponding build spec section
- [ ] Every KPI has a BI-layer calculation written in the platform's language
- [ ] Calculations reference curated model columns, not raw source tables
- [ ] Connection type and refresh schedule specified
- [ ] Tooltip specified for every chart
- [ ] Manual steps section present (even if empty)

## Lessons Learned

<!--
Append one bullet per lesson. Newest at the top. See `rules/lessons-learned-protocol.md`.
Format:
- **<YYYY-MM-DD> — <short title>.** Trigger: <project + failure pattern>. Change: <what changed>. learning_id: L-NNN
-->
