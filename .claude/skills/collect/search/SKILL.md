---
name: collect/search
description: Search the artifact library by keyword, topic tag, source type, or artifact_id. Returns ranked results with artifact_id, title, and a 1–2 sentence excerpt. Stays within context budget by reading CATALOG.md only — not artifact bodies.
inputs:
  - search query (natural language, topic tags, or artifact_id prefix)
  - optional filters: source_type, must_read, status
outputs:
  - ranked list of matching artifacts with paths (in-conversation response)
model_tier_hint: haiku
used_by_playbooks: [all — via /admin_resource search]
intent_tags: [search, find, lookup, artifact search, catalog search, retrieve]
---

# Skill: Search Artifact Library

## Boundaries

Does NOT: read full artifact bodies unless the user explicitly asks for one (that would bust context budget). Does NOT ingest new content. Does NOT modify CATALOG.md.

Returns artifact_ids and paths — the calling agent or user opens specific files themselves.

## Procedure

### Step 1 — Read CATALOG.md
Read `artifacts/CATALOG.md`. This is the only file read during search unless Step 4 triggers a deep read.

If CATALOG.md does not exist, report: "No artifacts have been ingested yet. Run `/admin_resource ingest` to add content."

### Step 2 — Parse the query
Classify the query into one or more match modes:

| Mode | Signal | Example |
|---|---|---|
| artifact_id lookup | Query starts with `ART-` | `ART-20260517-001` |
| topic tag match | Query matches known tags | `data-quality`, `kpi-definition` |
| title keyword match | Query words appear in title column | `retention policy` |
| source_type filter | Query specifies `confluence`, `jira`, `adhoc` | `source:confluence` |
| must_read filter | Query specifies `must_read:true` | `must_read` |
| status filter | Query specifies `status:superseded` | `status:current` |
| natural language | Anything else — match against title + topics | `what do we know about churn` |

Multiple modes can apply simultaneously.

### Step 3 — Score and rank matches

For each artifact row in CATALOG.md:
- +3 points: artifact_id exact match
- +2 points: topic tag exact match (per matching tag)
- +2 points: title contains all query keywords
- +1 point: title contains some query keywords
- +1 point: `must_read: true` (tie-breaker bonus)
- 0 points: `status: superseded` (include but rank below current)

Return top 10 results. If fewer than 3 results score >0, widen the search by stemming keywords.

### Step 4 — Format results
Return results in this format:

```
## Search Results for "<query>"
Found <N> matching artifacts (showing top <M>):

1. **ART-20260517-001** — Data Retention Policy Overview
   Source: confluence | Topics: data-retention, compliance, governance | must_read: false
   Path: artifacts/confluence/ART-20260517-001-data-retention-policy-overview.md

2. **ART-20260510-003** — KPI Governance Standards ⚠️ superseded
   Source: confluence | Topics: kpi-definition, governance | must_read: true
   Path: artifacts/confluence/ART-20260510-003-kpi-governance-standards.md
   Superseded by: ART-20260517-002

---
To read a specific artifact: "show me ART-20260517-001"
```

Mark superseded artifacts with ⚠️ so the user knows to prefer the replacement.

### Step 5 — Handle "show me" follow-up
If the user asks to see a specific artifact by ID, read that single file and return its full content. This is the only step that reads an artifact body.

### Step 6 — No results
If zero artifacts match:
```
No artifacts found matching "<query>".

Suggestions:
- Try broader keywords
- Check available topics: [list top 10 most frequent topic tags from CATALOG.md]
- Run /admin_resource ingest to add new content
```

## Self-Check Checklist
- [ ] Read CATALOG.md only (not artifact bodies) during Steps 1–4
- [ ] Results ranked by score, not arbitrary order
- [ ] Superseded artifacts flagged with ⚠️ and superseding artifact referenced
- [ ] Artifact path included in every result row
- [ ] No results message includes actionable suggestions

## Lessons Learned

<!--
Append one bullet per lesson. Newest at the top. See `rules/lessons-learned-protocol.md`.
Format:
- **<YYYY-MM-DD> — <short title>.** Trigger: <project + failure pattern>. Change: <what changed>. learning_id: L-NNN
-->
