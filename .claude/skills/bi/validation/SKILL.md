---
name: bi/validation
description: Verify dashboards and underlying models are functionally correct, usable, and data-accurate before release. Covers metric reconciliation, functional testing, and UX checks. Production-safety checks belong to generic/release-checklist.
inputs:
  - 09-build/09-build-spec.md
  - 02-kpi/02-kpi-dictionary.md
  - 04-discovery/04-source-map.md
outputs:
  - 10-validation/10-qa-report.md
model_tier_hint: sonnet
used_by_playbooks: [bi-dashboard]
intent_tags: [validation, qa, testing, reconciliation, data accuracy, functional test, uat]
---

# Skill: BI Validation & QA

## Boundaries

Does NOT: fix build issues (feeds back to `bi/dashboard-build`), check production infrastructure or access provisioning (that is `generic/release-checklist`). Validates correctness and usability — not production readiness.

Uses Snowflake MCP or other database MCP when available to run reconciliation queries directly.

## Three Validation Tracks

### Track 1 — Metric Reconciliation
Verify each KPI produces the correct number.

For each KPI:
1. Identify an independent reference source (prior report, finance export, known total)
2. Query the curated model directly for the same period
3. Compare — acceptable variance: ≤ 0.1% for financial metrics, ≤ 1% for behavioural metrics
4. Document result: match / variance / unexplained gap

If variance exceeds threshold: this is a reconciliation FAIL — block release, feed back to `dbt/model-build`.

### Track 2 — Functional Testing
Test every interactive element in the dashboard.

Checklist per page:
- [ ] All filters apply correctly (global filters affect all pages; local filters are scoped)
- [ ] Date range filter changes all time-series visuals
- [ ] Drill-throughs navigate to correct destination
- [ ] Cross-highlights work between visuals on the same page
- [ ] No broken visuals (blank charts, "data missing" errors)
- [ ] No performance issues (page load < 10 seconds on typical data volume)
- [ ] All KPI card values match the line chart values for the same period

### Track 3 — UX Acceptance
Verify the built dashboard matches the approved wireframe.

For each wireframe page:
- [ ] Same number and type of visuals as wireframe
- [ ] Same layout structure (row/column arrangement)
- [ ] Every KPI in MVP scope is visible
- [ ] Sample values in wireframe match approximate magnitude in live data
- [ ] Labels and titles match KPI canonical names from the dictionary

## Procedure

1. Run all three tracks
2. Record each finding as PASS, FAIL, or WARNING (minor — does not block release)
3. For every FAIL: document the specific issue and which step owns the fix
4. Overall verdict: PASS (all critical items pass) or FAIL (any critical item fails)

```markdown
---
step: 10
slug: <project-slug>
version: v1
created: <date>
reconciliation_result: PASS | FAIL
functional_result: PASS | FAIL
ux_result: PASS | FAIL
overall: PASS | FAIL
---

# QA Report: <Project Name>

## Overall: PASS | FAIL

## Track 1 — Metric Reconciliation
| KPI | Reference source | Model value | Reference value | Variance | Result |
|---|---|---|---|---|---|

## Track 2 — Functional Testing
| Test | Page | Result | Notes |
|---|---|---|---|

## Track 3 — UX Acceptance
| Check | Page | Result | Notes |
|---|---|---|---|

## Failures (must fix before release)
| ID | Track | Issue | Fix owner | Step to re-run |
|---|---|---|---|---|

## Warnings (can release with known limitation)
| ID | Issue | Impact | Acceptance rationale |
|---|---|---|---|
```

## Self-Check Checklist
- [ ] Every KPI reconciled against an independent reference
- [ ] Reconciliation variance within threshold (or documented exception)
- [ ] All three filter types tested (global, local, date)
- [ ] All drill-throughs tested
- [ ] Every wireframe page compared to built dashboard
- [ ] FAIL vs WARNING distinction applied (WARNING does not block release)
