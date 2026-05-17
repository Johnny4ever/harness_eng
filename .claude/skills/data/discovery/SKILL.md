---
name: data/discovery
description: Map each KPI to real data sources. Assess feasibility, identify grain and join paths, flag history depth limits and refresh constraints. De-risks the model design before any SQL is written.
inputs:
  - 02-kpi/02-kpi-dictionary.md
  - 03-alignment/03-alignment-summary.md (MVP scope)
  - artifacts/ (for any previously ingested source documentation)
outputs:
  - 04-discovery/04-source-map.md
model_tier_hint: sonnet
used_by_playbooks: [bi-dashboard, dbt-data-product, analysis-deep-dive]
intent_tags: [discovery, source mapping, feasibility, data source, grain, join, lineage]
---

# Skill: Data Discovery

## Boundaries

Does NOT: write SQL or dbt models (that is `dbt/model-build`), profile data quality in detail (that is `data/quality-profiling`), design the semantic model (that is `data/semantic-modeling`). Proposes candidate sources — does not confirm data quality.

Uses Snowflake MCP or other database MCP when available to inspect schemas directly. Falls back to artifact library and stakeholder-provided documentation when MCPs are unavailable.

## Procedure

### Step 1 — Map each KPI to a source
For every KPI in the MVP scope, identify:
- **Source system** — which system owns this data?
- **Candidate tables/views** — which specific objects contain the data?
- **Key fields** — which columns support the numerator, denominator, date, and filters?
- **Grain** — what does one row represent in the source?
- **Join path** — how do the tables connect to produce the required grain?

If a KPI cannot be mapped: record it as a **data gap** with a gap reason.

### Step 2 — Assess constraints for each source
For every source identified:
- **History depth** — how far back does data go? Does it meet the dashboard's required date range?
- **Refresh frequency** — how often is the source updated? Does it meet the dashboard's refresh requirement?
- **Access status** — is the data accessible now, or does access need to be requested?
- **PII / sensitivity** — any fields that require masking or restricted access?

### Step 3 — Classify feasibility per KPI

| Status | Meaning |
|---|---|
| ✅ Feasible | Source found, grain confirmed, join path clear, no blocking constraints |
| ⚠️ Conditional | Source found but has a constraint (history gap, refresh lag, PII, access pending) |
| ❌ Gap | No source found, or source exists but cannot support the KPI as defined |

### Step 4 — Flag source enablement needs
Any KPI with status ⚠️ or ❌ that requires cross-team action: list as a `source-enablement` item. The generator will trigger `generic/source-enablement` skill for these.

### Step 5 — Write the source map

```markdown
---
step: 04
slug: <project-slug>
version: v1
created: <date>
feasible_kpis: <N>
conditional_kpis: <N>
gap_kpis: <N>
---

# Source Map: <Project Name>

## KPI Feasibility Summary
| KPI | Status | Source system | Candidate table(s) | Grain | History | Refresh |
|---|---|---|---|---|---|---|

## KPI Detail

### <KPI Name>
**Status:** ✅ / ⚠️ / ❌
**Source:** <system>
**Tables:** <list>
**Key fields:** <numerator field>, <denominator field>, <date field>, <filter fields>
**Grain:** <one row per X>
**Join path:** <table A joins table B on key C>
**History depth:** <e.g. 3 years from 2022-01-01>
**Refresh:** <e.g. daily at 06:00 UTC>
**Constraints:** <any PII, access, or freshness issues>

## Data Gaps
| KPI | Gap reason | Impact | Recommended action |
|---|---|---|---|

## Source Enablement Required
| KPI | Action needed | Owner | Urgency |
|---|---|---|---|
```

## Self-Check Checklist
- [ ] Every MVP KPI has a row in the feasibility summary
- [ ] Every feasible/conditional KPI has grain documented
- [ ] Every feasible/conditional KPI has join path documented (not just table names)
- [ ] Every gap has a gap reason (not just "no data found")
- [ ] History depth checked against the dashboard's required date range
- [ ] Source enablement table populated for all ⚠️ and ❌ items

## Lessons Learned

<!--
Append one bullet per lesson. Newest at the top. See `rules/lessons-learned-protocol.md`.
Format:
- **<YYYY-MM-DD> — <short title>.** Trigger: <project + failure pattern>. Change: <what changed>. learning_id: L-NNN
-->
