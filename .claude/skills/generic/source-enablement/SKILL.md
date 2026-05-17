---
name: generic/source-enablement
description: Unblock data access and manage upstream dependencies when discovery flags source gaps, missing fields, or cross-team data availability issues.
inputs:
  - 04-discovery/04-source-map.md (source enablement items)
  - 05-profiling/05-quality-report.md (blocking issues, if available)
outputs:
  - 13-source-enablement/13-enablement-tracker.md
model_tier_hint: sonnet
used_by_playbooks: [bi-dashboard, dbt-data-product]
intent_tags: [source enablement, data access, upstream dependency, field contract, access request, blocker]
---

# Skill: Source Enablement

## Boundaries

Does NOT: build transformations, assess data quality, or design models. Works at the boundary between the BI team and source system owners to unblock data pipeline access.

This is a cross-cutting skill — it can be triggered at any checkpoint when a discovery or profiling output flags an unresolved data access issue.

## Procedure

### Step 1 — Triage enablement items
From the source map's "Source Enablement Required" table and the profiling report's "Blocking Issues" table, classify each item:

| Type | Description | Typical resolution time |
|---|---|---|
| Access request | Team/role needs read access to a table | 1–5 days |
| Field contract | Upstream team must add/expose a field | 1–4 weeks |
| Schema change | Source table structure needs to change | 2–8 weeks |
| New data feed | Entirely new source system integration | 4–12 weeks |

### Step 2 — Draft access requests
For each access request item, produce a draft request message containing:
- Which table/schema/database needs access
- Which service account or user group needs it
- Business justification (one sentence)
- Which KPI is blocked
- Urgency (blocking release / non-blocking)

### Step 3 — Document field contracts
For each field contract item:
- What field is needed (name, data type, business meaning)
- Which source table it should be added to
- Which upstream team owns that table
- Sample values or calculation logic if the field must be derived
- SLA needed (before which delivery milestone)

### Step 4 — Build the tracker

```markdown
---
step: 13
slug: <project-slug>
version: v1
created: <date>
open_items: <N>
blocking_items: <N>
---

# Source Enablement Tracker: <Project Name>

## Summary
<N> enablement items. <N> are blocking release. <N> are non-blocking.

## Items

### ENB-001: <short description>
**Type:** Access request / Field contract / Schema change / New feed
**KPI blocked:** <name>
**Blocking release?** Yes / No
**Source table:** <schema.table>
**Upstream owner:** <team or contact>
**Action required:** <specific ask>
**Status:** Open / In progress / Resolved
**Target resolution:** <date>
**Draft request:**
> <ready-to-send message for the data owner>

## Resolved Items
| ID | Item | Resolved date | How resolved |
|---|---|---|---|

## Impact on Delivery
| KPI | Enablement item | If unresolved by <date>: impact |
|---|---|---|
```

## Self-Check Checklist
- [ ] Every ⚠️ and ❌ item from the source map has a tracker entry
- [ ] Every blocking item has a target resolution date
- [ ] Every item has a draft request message ready to send
- [ ] Impact on delivery documented for all blocking items
- [ ] Non-blocking items clearly separated from blocking ones
