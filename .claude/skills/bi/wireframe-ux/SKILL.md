---
name: bi/wireframe-ux
description: Translate KPIs and business questions into a dashboard wireframe with page layout, visual types, sample data, and interaction design. De-risks UX before build.
inputs:
  - 04-discovery/04-source-map.md (primary — grain and available columns drive what can be shown)
  - 02-kpi/02-kpi-dictionary.md
  - 03-alignment/03-alignment-summary.md (MVP scope)
outputs:
  - 08-wireframe/08-wireframe.md
model_tier_hint: sonnet
used_by_playbooks: [bi-dashboard]
intent_tags: [wireframe, ux, layout, dashboard design, visual, chart, mockup, prototype]
---

# Skill: Wireframe & UX Design

## Boundaries

Does NOT: define KPIs (consumes them as-is), assess data feasibility (consumes discovery output), implement BI-layer calculations or DAX/SQL, make scope decisions.

The discovery output is the **primary input** — wireframes must reflect only columns and grains confirmed as available. Do not design visuals for data that is not in the source map.

Uses Figma MCP when available for live prototype output. Falls back to structured Markdown wireframes when Figma is unavailable.

## Procedure

### Step 1 — Establish the page structure
Based on the audience and business objective, decide:
- How many dashboard pages/tabs?
- What is the primary question each page answers?
- What is the navigation pattern (tabs, drill-through, bookmark)?

Rule: one clear question per page. If a stakeholder wants "everything on one page," split it anyway and present the case for multi-page.

### Step 2 — Design each page
For each page:

**Layout block (describe in text):**
```
[Page name] — answers: <primary question>

Row 1: KPI cards  — MRR | Churn Rate | Conversion Rate
Row 2: [Line chart: MRR trend 12M]  |  [Bar chart: Churn by cohort]
Row 3: [Table: Customer detail with drill-through to customer page]

Filters (header): Date range | Region | Product tier
```

**Visual specification per chart:**
| Visual | KPI/field | Chart type | Grain shown | Sample value |
|---|---|---|---|---|
| MRR trend | `monthly_recurring_revenue` | Line | Monthly | $124,000 |
| Churn cohort | `churn_rate` | Clustered bar | Monthly cohort | 2.4% |

Derive sample values from the source map's row count, known ranges, or reasonable estimates — label them "illustrative."

### Step 3 — Specify interactions
For each page: list filters, drill-throughs, cross-highlights, and tooltips.

**Filter behaviour:**
- Which filters are global (all pages)?
- Which are local (this page only)?
- What is the default state?

### Step 4 — Flag UX risks
List any design choices that depend on data not yet confirmed, and any layout decisions that need stakeholder validation.

### Step 5 — Write the wireframe document

```markdown
---
step: 08
slug: <project-slug>
version: v1
created: <date>
pages: <N>
figma_url: <url or "not available">
---

# Wireframe: <Project Name>

## Page Index
| Page | Primary question | KPIs shown |
|---|---|---|

## Page Designs
### Page 1: <Name>
<layout description>
<visual specification table>
<filter and interaction spec>

## Sample Data Notes
<how sample values were derived, label as illustrative>

## UX Risks & Open Items
| Risk | Impact | Resolution needed from |
|---|---|---|

## VERSION-INDEX
| Version | Date | Change |
|---|---|---|
```

## Self-Check Checklist
- [ ] Every KPI in MVP scope appears on at least one page
- [ ] Every visual references only columns confirmed in the source map
- [ ] Sample values are present for every chart (even if illustrative)
- [ ] Filter scope (global vs local) specified for every filter
- [ ] No visual designed for a ❌ gap KPI from discovery
- [ ] UX risks section present (even if empty — note "no risks identified")

## Lessons Learned

<!--
Append one bullet per lesson. Newest at the top. See `rules/lessons-learned-protocol.md`.
Format:
- **<YYYY-MM-DD> — <short title>.** Trigger: <project + failure pattern>. Change: <what changed>. learning_id: L-NNN
-->
