# Skills Catalog

Generator reads this file FIRST before loading any skill.
Resolution order: native plugin skill > local skill > escalate to user.

**Maintenance rule:** any commit that adds, renames, or removes a skill MUST update this file in the same commit.

---

## Local Skills

| Path | Category | Description | Intent tags | Used by playbooks |
|---|---|---|---|---|
| **BI** | | | | |
| `bi/kpi-definition` | bi | Convert business language into precise KPI definitions with grain, filters, date logic, ownership | kpi, metric, definition, kpi dictionary, business measure | bi-dashboard, kpi-proof |
| `bi/wireframe-ux` | bi | Translate KPIs into dashboard wireframe — pages, visuals, sample data, interactions | wireframe, ux, layout, dashboard design, chart, mockup | bi-dashboard |
| `bi/dashboard-build` | bi | Implement approved wireframe as working dashboard — BI-layer calculations, visual specs | dashboard build, power bi, tableau, looker, dax, publish | bi-dashboard |
| `bi/validation` | bi | Metric reconciliation, functional testing, UX acceptance — grades correctness before release | validation, qa, reconciliation, data accuracy, functional test | bi-dashboard |
| **Data** | | | | |
| `data/discovery` | data | Map KPIs to sources — grain, join path, history depth, refresh constraints, gap classification | discovery, source mapping, feasibility, grain, join, lineage | bi-dashboard, dbt-data-product, analysis-deep-dive |
| `data/quality-profiling` | data | Profile sources across 6 dimensions — null rates, freshness, uniqueness, validity, remediation | data quality, profiling, completeness, null rate, freshness, fitness | bi-dashboard, dbt-data-product |
| `data/semantic-modeling` | data | Design fact/dimension model — grain, relationships, KPI-to-model mapping | semantic model, dimensional model, fact table, dimension, star schema | bi-dashboard, dbt-data-product |
| **dbt** | | | | |
| `dbt/model-build` | dbt | Build staged/mart SQL models with dbt conventions, CTE patterns, schema.yml tests | sql, dbt, transformation, staging, mart, cte, model build | bi-dashboard, dbt-data-product |
| **Generic** | | | | |
| `generic/requirement-intake` | generic | Transform unstructured requests into structured requirement document | requirement, intake, brief, stakeholder request, scope | bi-dashboard, dbt-data-product, analysis-deep-dive |
| `generic/stakeholder-alignment` | generic | Stabilise scope — MVP vs Phase 2, conflict resolution, decisions, signoffs | alignment, stakeholder, scope, mvp, prioritise, signoff | bi-dashboard, dbt-data-product |
| `generic/documentation` | generic | Compile knowledge pack — KPI dictionary, dashboard guide, data lineage, runbook | documentation, knowledge pack, handover, runbook, lineage | bi-dashboard, dbt-data-product |
| `generic/governance-check` | generic | Check enterprise KPI registry, enforce naming standards, flag reuse candidates | governance, reuse, naming, enterprise, duplicate, kpi registry | bi-dashboard, dbt-data-product |
| `generic/source-enablement` | generic | Track and draft access requests, field contracts for blocked data sources | source enablement, access request, field contract, blocker | bi-dashboard, dbt-data-product |
| `generic/release-checklist` | generic | Production go-live — environment, refresh, access, smoke test, rollback plan | release, deploy, production, publish, access, refresh, rollback | bi-dashboard, dbt-data-product |
| **Eval** | | | | |
| `eval/contract-definition` | eval | Write sprint contracts with observable binary acceptance criteria before a checkpoint | contract, criteria, acceptance, define done, before checkpoint | all |
| `eval/output-grading` | eval | Grade generator deliverables PASS/FAIL with hard thresholds — skeptical by default | grade, evaluate, verdict, assess, quality check, pass fail | all |
| `eval/feedback-synthesis` | eval | Convert FAIL verdict into specific actionable rework brief — no praise, no softening | feedback, rework, correction, fail, iteration | all |
| `eval/retrospective` | eval | Post-delivery lesson-learned artifact covering iteration patterns and improvement recommendations | retrospective, lesson learned, review, post-delivery, improve | all |
| **Collect** | | | | |
| `collect/ingest-confluence` | collect | Fetch a Confluence page via MCP and decompose into topic-based artifact files with YAML front-matter | confluence, ingest, wiki, page, knowledge, documentation | all — via /admin_resource ingest |
| `collect/ingest-jira` | collect | Fetch Jira issues or JQL results via MCP and decompose into topic-based artifact files | jira, ingest, ticket, issue, epic, story, requirements | all — via /admin_resource ingest |
| `collect/ingest-adhoc` | collect | Decompose pasted content (emails, Slack, meeting notes, docs) into artifact files | adhoc, ingest, paste, email, slack, meeting notes | all — via /admin_resource ingest |
| `collect/catalog-index` | collect | Update CATALOG.md and CHANGELOG.md after new artifacts are written | catalog, index, catalog-index, artifact registry, changelog | all — via /admin_resource ingest |
| `collect/version-supersede` | collect | Mark an older artifact as superseded when new content replaces it | supersede, version, artifact version, replace, changelog | all — via /admin_resource ingest |
| `collect/search` | collect | Search the artifact library by keyword, topic, source type, or artifact_id — reads CATALOG.md only | search, find, lookup, artifact search, catalog search | all — via /admin_resource search |

---

## Native Skills (Claude Code Plugins)

Prefer native over local when they cover the same need. The generator checks this section before the local skills section.

| Native name | Use when | Overlaps with local skill |
|---|---|---|
| `anthropic-skills:docx` | Producing polished Word document deliverables | `generic/documentation` (when available) |
| `anthropic-skills:pdf` | Reading PDFs as source input or producing PDF reports | — |
| `anthropic-skills:xlsx` | Reading or producing spreadsheet deliverables | — |
| `anthropic-skills:pptx` | Producing slide deck deliverables | — |
| `engineering:documentation` | Writing technical documentation, READMEs, runbooks | `generic/documentation` (when available) |
| `engineering:code-review` | Reviewing SQL, dbt, or Python code quality | — |
| `engineering:system-design` | Architecture and system design diagrams | — |
| `engineering:debug` | Diagnosing errors in SQL models or Python code | — |
| `engineering:testing-strategy` | Writing test strategies for dbt or application code | `dbt/test-design` (when available) |
| `atlassian:capture-tasks-from-meeting-notes` | Extracting structured tasks from meeting transcripts during collection | — |
| `atlassian:triage-issue` | Triaging and categorising Jira issues during collection | — |
| `atlassian:search-company-knowledge` | Searching Confluence/Jira for existing definitions | — |
| `atlassian:generate-status-report` | Producing project status reports | — |

---

## Resolution Algorithm

When the generator needs a skill for a step:

```
1. Does the playbook step specify an explicit skill name or native skill name?
     YES → use it exactly as specified
     NO  → continue to step 2

2. Match the step's intent against the "Intent tags" column in Local Skills above.
     MATCH in native section → check if native skill is available in this session
       AVAILABLE → use native skill
       NOT AVAILABLE → fall through to local
     MATCH in local section → load .claude/skills/<path>/SKILL.md
     NO MATCH → go to step 3

3. No skill found.
   → Write to STATUS.md: step blocked — no skill match
   → Report to user: "No skill found for <step>. Available skills: [list]. 
     Please specify a skill or provide context."
   → STOP — do not attempt the step
```

**Tie-breaking:** when both native and local match, the playbook front-matter can specify `prefer: native` (default) or `prefer: local`. Default is native.

---

## Skills Pending (added in future phases)

| Path | Phase | Status |
|---|---|---|
| `dbt/test-design` | Phase 6 | ⏳ pending |
| `dbt/documentation` | Phase 6 | ⏳ pending |
