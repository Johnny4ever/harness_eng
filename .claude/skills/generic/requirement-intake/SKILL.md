---
name: generic/requirement-intake
description: Transform unstructured stakeholder requests into a clean, structured requirement document that downstream steps can rely on without re-interpreting scattered notes.
inputs:
  - raw stakeholder input (tickets, emails, meeting notes, Confluence links, chat logs)
  - existing dashboards or report references (optional)
  - business glossaries (optional)
outputs:
  - 01-requirement/01-requirement.md
model_tier_hint: sonnet
used_by_playbooks: [bi-dashboard, dbt-data-product, analysis-deep-dive]
intent_tags: [requirement, intake, brief, stakeholder request, scope, user story]
---

# Skill: Requirement Intake

## Boundaries

Does NOT: define KPI calculation logic (that is `bi/kpi-definition`), assess data feasibility (that is `data/discovery`), scope the MVP (that is `generic/stakeholder-alignment`), design visuals (that is `bi/wireframe-ux`).

## Procedure

### Step 1 — Parse all inputs
Read every source provided. Identify: who asked, what they want to see, who the audience is, what decisions this output supports, what the target platform is (if stated), and any deadlines.

### Step 2 — Extract structured fields
Populate each field below. If a field cannot be answered from the input, mark it `[OPEN — needs clarification]` and add to the clarification log.

**Required fields:**
- **Business objective** — one sentence: what decision or action does this enable?
- **Primary audience** — who reads this and how often?
- **Requested KPIs / metrics** — names only, no definitions (definitions belong to `bi/kpi-definition`)
- **Requested dimensions / filters** — how users want to slice the data
- **Target BI platform** — Power BI / Tableau / Looker / other
- **Refresh frequency** — real-time / hourly / daily / weekly
- **Data scope** — geography, time range, product lines, entity types
- **Known data sources** — any systems mentioned by the stakeholder
- **Out of scope** — what is explicitly excluded
- **Success criteria** — how the stakeholder will judge if this is "done"

### Step 3 — Front-load clarification
Before writing the final document, list every field marked `[OPEN]`. If the user is available, ask all clarification questions in a single grouped message — not one at a time. If the user is not available, make a reasonable assumption, document it as an assumption, and flag it for validation at stakeholder alignment.

### Step 4 — Write the requirement document

```markdown
---
step: 01
slug: <project-slug>
version: v1
created: <date>
open_items: <N>
---

# Requirement Document: <Project Name>

## Business Objective
<one sentence>

## Audience
<who, frequency, use case>

## Requested KPIs
| KPI name | Description (business language only) |
|---|---|

## Dimensions & Filters
<list>

## Platform & Refresh
Platform: <name>  |  Refresh: <frequency>

## Data Scope
<geography, time range, entity types>

## Known Data Sources
<list — unvalidated>

## Out of Scope
<list>

## Success Criteria
<how stakeholder judges done>

## Assumptions & Clarifications
| ID | Item | Assumption made | Must validate with |
|---|---|---|---|
```

## Self-Check Checklist
- [ ] Business objective is one sentence and action-oriented
- [ ] Every requested KPI is named (no definitions written here)
- [ ] Every `[OPEN]` field has a corresponding assumption or clarification request
- [ ] Out of scope section is non-empty
- [ ] Success criteria is stated in stakeholder's language, not technical language

## Lessons Learned

<!--
Append one bullet per lesson. Newest at the top. See `rules/lessons-learned-protocol.md`.
Format:
- **<YYYY-MM-DD> — <short title>.** Trigger: <project + failure pattern>. Change: <what changed>. learning_id: L-NNN
-->
