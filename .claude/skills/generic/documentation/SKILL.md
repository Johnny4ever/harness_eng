---
name: generic/documentation
description: Compile a complete, handover-ready knowledge pack covering all delivery artefacts, decisions, KPI definitions, model lineage, and operational runbook. Ensures the dashboard is understandable, supportable, and reusable.
inputs:
  - all prior step outputs (reads STATUS.md to discover what was produced)
  - DECISIONS.md
outputs:
  - 11-documentation/11-knowledge-pack.md
model_tier_hint: sonnet
used_by_playbooks: [bi-dashboard, dbt-data-product]
intent_tags: [documentation, knowledge pack, handover, runbook, lineage, data dictionary]
---

# Skill: Documentation & Knowledge Pack

## Boundaries

Does NOT: re-derive definitions or re-run analysis. Synthesises and organises what was already produced. If a prior step's output is missing, flags it as a gap rather than recreating it.

Prefer native skill `engineering:documentation` for technical-only documentation tasks. Use this skill when the output needs to cover both business context (KPI definitions, stakeholder decisions) and technical content (lineage, model descriptions, runbook).

## Procedure

### Step 1 — Inventory all artefacts
Read `STATUS.md` to get the list of all completed steps and their output paths. Do not read each file — just confirm they exist and note the path.

### Step 2 — Compile the knowledge pack sections

**Section 1 — Project Overview**
- Business objective (from requirement doc)
- Primary audience and use cases
- What was built and what was deferred (from alignment summary)
- Key decisions made during delivery (from DECISIONS.md)

**Section 2 — KPI Dictionary (summary)**
- Reproduce the KPI records in a simplified format suitable for business users
- Include: name, business description, calculation (plain English), owner, update frequency
- Do NOT include technical grain/join details here — those go in Section 4

**Section 3 — Dashboard Guide**
- Page-by-page description: what each page shows and when to use it
- How to use key filters
- How to interpret each KPI card (what is a "good" value?)
- Drill-through paths

**Section 4 — Data Lineage**
- Source → staging → mart lineage for each KPI
- Which raw tables feed which curated models
- Transformation logic summary (business-language description of each model's purpose)
- Known data quality caveats from the profiling report

**Section 5 — Operational Runbook**
- Refresh schedule and what triggers it
- Who to contact if the dashboard shows stale data
- Who to contact if a KPI value looks wrong
- How to request a new metric or filter
- Rollback procedure (reference to release checklist)

### Step 3 — Write the knowledge pack

```markdown
---
step: 11
slug: <project-slug>
version: v1
created: <date>
---

# Knowledge Pack: <Project Name>

## 1. Project Overview
...

## 2. KPI Dictionary
| KPI | Plain-English description | Calculation | Owner | Updates |
|---|---|---|---|---|

## 3. Dashboard Guide
### Page: <Name>
...

## 4. Data Lineage
| KPI | Raw source | Staging model | Mart model | Key transformations |
|---|---|---|---|---|

## 5. Operational Runbook
...
```

## Self-Check Checklist
- [ ] All 5 sections present
- [ ] Every KPI from the MVP scope appears in Section 2
- [ ] Section 3 covers every dashboard page
- [ ] Section 4 shows lineage for every KPI (not just some)
- [ ] Runbook has named contacts (not just "the team")
- [ ] No technical jargon in Sections 1–3 that a business user would not understand

## Lessons Learned

<!--
Append one bullet per lesson. Newest at the top. See `rules/lessons-learned-protocol.md`.
Format:
- **<YYYY-MM-DD> — <short title>.** Trigger: <project + failure pattern>. Change: <what changed>. learning_id: L-NNN
-->
