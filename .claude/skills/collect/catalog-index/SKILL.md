---
name: collect/catalog-index
description: Update CATALOG.md and CHANGELOG.md after new artifacts are written. Maintains the master artifact index so other agents can search efficiently.
inputs:
  - list of newly written artifact files (from ingest skills)
  - artifacts/CATALOG.md (existing)
  - artifacts/CHANGELOG.md (existing)
outputs:
  - artifacts/CATALOG.md (updated)
  - artifacts/CHANGELOG.md (updated)
model_tier_hint: haiku
used_by_playbooks: [all — via /admin_resource ingest]
intent_tags: [catalog, index, catalog-index, artifact registry, changelog]
---

# Skill: Catalog Index

## Boundaries

Does NOT: ingest source content (that is `ingest-*` skills), manage supersession chains (that is `collect/version-supersede`), search the catalog (that is `collect/search`).

This skill runs after every ingest batch. It is lightweight — it reads only the front-matter of new artifacts, not their full bodies.

## CATALOG.md Structure

```markdown
# Artifact Catalog

Last updated: <YYYY-MM-DD HH:MM>
Total artifacts: <N>

## Index

| artifact_id | title | source_type | topics | must_read | status | created |
|---|---|---|---|---|---|---|
| ART-20260517-001 | Data retention policy overview | confluence | data-retention, compliance, governance | false | current | 2026-05-17 |
```

The table is sorted by `artifact_id` descending (newest first).

## CHANGELOG.md Structure

```markdown
# Artifact Changelog

| Date | Action | artifact_id | Title | Notes |
|---|---|---|---|---|
| 2026-05-17 | Added | ART-20260517-001 | Data retention policy overview | Ingested from Confluence |
| 2026-05-17 | Superseded | ART-20260510-003 | Old KPI definitions | Superseded by ART-20260517-002 |
```

## Procedure

### Step 1 — Read front-matter of new artifacts
For each file path in the ingest output:
- Read only the YAML front-matter block (lines between `---` markers)
- Extract: artifact_id, title, source_type, topics, must_read, status, created

Do NOT read the full artifact body — context budget applies.

### Step 2 — Update CATALOG.md

1. Read the current CATALOG.md
2. Append new rows to the index table (one row per new artifact)
3. Re-sort by artifact_id descending
4. Update `Total artifacts` count and `Last updated` timestamp
5. Write the updated file

### Step 3 — Update CHANGELOG.md

1. Read the current CHANGELOG.md
2. Prepend new rows (newest at top):
   - For each new artifact: `Action = Added`
   - `Notes` = brief source description (e.g. "Ingested from Confluence: <source_title>")
3. Write the updated file

### Step 4 — Initialize if missing
If CATALOG.md or CHANGELOG.md do not exist, create them with the header structure above before inserting rows.

### Step 5 — Report
Return:
```
Catalog updated: <N> artifacts added. Total: <M> artifacts indexed.
```

## Self-Check Checklist
- [ ] Every new artifact appears as a row in CATALOG.md
- [ ] Total artifacts count is accurate
- [ ] `Last updated` timestamp is today's date
- [ ] CHANGELOG.md has one row per new artifact with `Added` action
- [ ] Table is sorted newest-first in CATALOG.md
- [ ] Did NOT read full artifact bodies (front-matter only)
