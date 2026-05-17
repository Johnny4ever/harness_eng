---
name: generic/release-checklist
description: Manage the transition from build-complete to production-live. Covers environment mapping, refresh setup, access provisioning, smoke tests, and rollback plans. Production-safety only — does not grade quality (that is bi/validation).
inputs:
  - 10-validation/10-qa-report.md (must be PASS before release)
  - 09-build/09-build-spec.md
outputs:
  - 12-release/12-release-checklist.md
model_tier_hint: sonnet
used_by_playbooks: [bi-dashboard, dbt-data-product]
intent_tags: [release, deploy, production, publish, access, refresh, rollback, go-live]
---

# Skill: Release Checklist

## Boundaries

Does NOT: re-validate data quality or metric accuracy (that is `bi/validation`), write SQL or build dashboards. Operationalises what QA has confirmed is correct.

**Hard prerequisite:** QA report must show `overall: PASS` before this skill runs. If QA is FAIL, this skill must not proceed.

## Procedure

### Step 1 — Verify prerequisites
- [ ] `10-validation/10-qa-report.md` exists with `overall: PASS`
- [ ] All FAIL items from QA are resolved (not just acknowledged)
- [ ] Build specification is final and matches what was QA'd

### Step 2 — Environment checklist
For each environment (dev → staging → production):
- Data connections point to the correct environment database/schema
- Credentials and service accounts are production-grade (not developer accounts)
- Row-level security rules are applied and tested with a production user account
- Dashboard is published to the correct workspace/project/site

### Step 3 — Refresh setup
- Scheduled refresh is configured in the BI platform
- Refresh schedule matches the requirement document
- Refresh failure alerting is configured (who gets notified?)
- First manual refresh after deployment completed and verified

### Step 4 — Access provisioning
- Correct user groups or roles have been granted access
- Access is validated by logging in as a representative end user (not admin)
- Data governance team notified if new datasets were published

### Step 5 — Smoke test (post-deploy)
After promoting to production:
- [ ] Dashboard loads without errors in production environment
- [ ] KPI card values are non-zero and in expected range
- [ ] Date filter defaults to expected range
- [ ] At least one drill-through navigates correctly
- [ ] Scheduled refresh runs successfully at least once

### Step 6 — Rollback plan
Document the rollback procedure:
- How to revert to the previous dashboard version if issues are found post-launch
- Which dbt models were changed and how to roll back (git revert + dbt run)
- Who authorises a rollback and what the communication protocol is

### Step 7 — Write the release checklist

```markdown
---
step: 12
slug: <project-slug>
version: v1
created: <date>
qa_report: PASS
environment: production
release_status: ready | released | rolled-back
---

# Release Checklist: <Project Name>

## Prerequisites
- [x] QA report: PASS (dated <date>)
- [x] All QA FAIL items resolved

## Environment
| Check | Status | Notes |
|---|---|---|
| Production connection | ✅ / ❌ | |
| Service account credentials | ✅ / ❌ | |
| RLS applied | ✅ / ❌ | |
| Published to correct workspace | ✅ / ❌ | |

## Refresh Setup
Scheduled: <frequency> at <time UTC>
Failure alert: <who / how>
First manual refresh: ✅ / ❌

## Access Provisioning
| Group / Role | Access granted | Validated |
|---|---|---|

## Smoke Test Results
| Test | Result |
|---|---|
| Dashboard loads | ✅ / ❌ |
| KPI values in range | ✅ / ❌ |
| Date filter default | ✅ / ❌ |
| Drill-through | ✅ / ❌ |
| First scheduled refresh | ✅ / ❌ |

## Rollback Plan
<step-by-step rollback procedure>
<authorisation: who decides, how communicated>
```

## Self-Check Checklist
- [ ] QA PASS confirmed before writing any of this document
- [ ] Production vs staging connections verified (not assumed)
- [ ] Refresh failure alerting has a named recipient
- [ ] Access validated as end user (not admin)
- [ ] Rollback plan is actionable (not "revert the changes")

## Lessons Learned

<!--
Append one bullet per lesson. Newest at the top. See `rules/lessons-learned-protocol.md`.
Format:
- **<YYYY-MM-DD> — <short title>.** Trigger: <project + failure pattern>. Change: <what changed>. learning_id: L-NNN
-->
