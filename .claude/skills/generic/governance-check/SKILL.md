---
name: generic/governance-check
description: Check whether KPIs, dimensions, and models already exist in the enterprise, enforce naming standards, and flag project-specific logic that should be elevated to shared assets.
inputs:
  - 02-kpi/02-kpi-dictionary.md
  - 06-semantic-model/06-semantic-model.md
  - artifacts/ (enterprise KPI registry and model catalogue if ingested)
outputs:
  - 14-governance/14-governance-report.md
model_tier_hint: sonnet
used_by_playbooks: [bi-dashboard, dbt-data-product]
intent_tags: [governance, reuse, naming standard, enterprise, duplicate, shared asset, kpi registry]
---

# Skill: Governance & Reuse Check

## Boundaries

Does NOT: define KPIs (consumes them), build models, make build/no-build decisions. Audits, compares, and recommends — the project team decides what to do with the findings.

## Procedure

### Step 1 — Check for existing enterprise KPIs
Search the artifact library (`/admin_resource search "kpi <name>"`) for each KPI in the dictionary. For every match:
- Is the enterprise definition the same as the project definition?
- If different: is the project definition a specialisation (narrower scope) or a conflict (different formula)?

Record as: ✅ Aligned / ⚠️ Specialisation / ❌ Conflict

### Step 2 — Check for existing models
Search for existing dbt models or BI datasets that overlap with the project's semantic model:
- Is there an existing `fct_orders` or `dim_customer`?
- If yes: can the project reuse it, or does the grain/scope differ?

Record as: ✅ Reuse / ⚠️ Extend / ❌ Duplicate (must consolidate)

### Step 3 — Enforce naming standards
Check every KPI canonical name and model name against the enterprise naming convention:
- snake_case for all identifiers
- `fct_` prefix for fact tables, `dim_` for dimensions, `stg_` for staging
- KPI names follow `<noun>_<metric_type>` pattern (e.g. `monthly_recurring_revenue`, not `MRR` or `mrr_monthly`)

Flag every violation with the corrected name.

### Step 4 — Flag promotion candidates
Identify project-specific logic that should become a shared enterprise asset:
- A new KPI definition that would benefit other teams
- A new dimension that is domain-agnostic and reusable
- A staging model for a source system not previously modelled

### Step 5 — Write the governance report

```markdown
---
step: 14
slug: <project-slug>
version: v1
created: <date>
---

# Governance Report: <Project Name>

## KPI Alignment
| KPI | Enterprise match | Status | Action required |
|---|---|---|---|
| `monthly_recurring_revenue` | Yes — revenue.kpi_registry | ✅ Aligned | None |
| `trial_churn_rate` | No match | 🆕 New | Consider registering |

## Model Reuse
| Model | Existing asset | Status | Action required |
|---|---|---|---|
| `fct_subscriptions` | None | 🆕 New | Build as project model |
| `dim_customer` | `core.dim_customer` | ✅ Reuse | Reference existing |

## Naming Violations
| Item | Current name | Corrected name | Where to fix |
|---|---|---|---|

## Promotion Candidates (recommend elevating to shared)
| Asset | Type | Rationale |
|---|---|---|

## Actions Required Before Release
| Priority | Action | Owner |
|---|---|---|
| High | Resolve ❌ KPI conflicts with central BI team | Project lead |
| Medium | Register new KPIs in enterprise registry | Data governance |
```

## Self-Check Checklist
- [ ] Every KPI in the dictionary checked against enterprise registry
- [ ] Every model in the semantic design checked for existing duplicates
- [ ] Every naming violation listed with corrected name
- [ ] Promotion candidates section present (even if empty)
- [ ] Actions are prioritised (High / Medium / Low)

## Lessons Learned

<!--
Append one bullet per lesson. Newest at the top. See `rules/lessons-learned-protocol.md`.
Format:
- **<YYYY-MM-DD> — <short title>.** Trigger: <project + failure pattern>. Change: <what changed>. learning_id: L-NNN
-->
