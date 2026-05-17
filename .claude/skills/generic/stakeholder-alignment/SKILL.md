---
name: generic/stakeholder-alignment
description: Stabilise scope and expectations by surfacing conflicts between stakeholder perspectives, establishing MVP boundaries, and tracking decisions and signoffs before heavy build work proceeds.
inputs:
  - 01-requirement/01-requirement.md
  - 02-kpi/02-kpi-dictionary.md (if available)
  - data/discovery output (if available — for feasibility constraints)
outputs:
  - 03-alignment/03-alignment-summary.md
model_tier_hint: sonnet
used_by_playbooks: [bi-dashboard, dbt-data-product]
intent_tags: [alignment, stakeholder, scope, mvp, prioritise, signoff, conflict]
---

# Skill: Stakeholder Alignment

## Boundaries

Does NOT: define KPIs (that is `bi/kpi-definition`), assess data feasibility (that is `data/discovery`), build deliverables.

The output of this skill is a **decision record and scope boundary** — not a design.

## Procedure

### Step 1 — Identify potential conflicts
Read the requirement document and KPI dictionary (if available). Look for:
- KPIs that different stakeholders may define differently
- Scope items that are ambiguous (in or out?)
- Features that may be technically infeasible based on discovery findings
- Competing priorities between audience groups

### Step 2 — Define MVP boundary
Given the full requested scope, propose an MVP that:
- Delivers the highest-value KPIs for the primary audience
- Excludes items that are high effort, low impact, or blocked by data
- Can be delivered in a single build cycle
- Has a clear "Phase 2" backlog for deferred items

Present MVP vs Phase 2 split as a table.

### Step 3 — Surface decisions required
List every open question that requires a stakeholder decision before build can proceed. Frame each as a specific decision, not a vague question.

### Step 4 — Write the alignment summary

```markdown
---
step: 03
slug: <project-slug>
version: v1
created: <date>
---

# Alignment Summary: <Project Name>

## Agreed Scope — MVP

| Item | Type | Priority | Notes |
|---|---|---|---|
| <KPI or feature> | KPI / dimension / filter | Must-have / Nice-to-have | |

## Deferred to Phase 2

| Item | Reason deferred |
|---|---|

## Decisions Made

| ID | Decision | Rationale | Owner | Date |
|---|---|---|---|---|

## Decisions Required (blocking)

| ID | Question | Options | Who decides | Deadline |
|---|---|---|---|---|

## Conflicts Identified

| ID | Conflict | Resolution |
|---|---|---|

## Signoff Status

| Stakeholder | Role | Status | Date |
|---|---|---|---|
| | | Pending / Approved | |
```

## Self-Check Checklist
- [ ] MVP boundary is explicit — every requested item is either In or Deferred
- [ ] Every deferred item has a stated reason
- [ ] Every blocking decision has an owner and a deadline
- [ ] Conflicts section is present (even if empty — note "no conflicts identified")
- [ ] Signoff table lists at least the primary stakeholder

## Lessons Learned

<!--
Append one bullet per lesson. Newest at the top. See `rules/lessons-learned-protocol.md`.
Format:
- **<YYYY-MM-DD> — <short title>.** Trigger: <project + failure pattern>. Change: <what changed>. learning_id: L-NNN
-->
