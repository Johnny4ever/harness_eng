---
name: collect/ingest-confluence
description: Fetch a Confluence page via MCP and decompose it into one or more discrete, topic-based artifact files with structured YAML front-matter.
inputs:
  - Confluence page URL
  - next artifact_id sequence number (from collector agent)
  - source_type: confluence
outputs:
  - artifacts/confluence/ART-<date>-<NNN>-<slug>.md (one per topic)
model_tier_hint: sonnet
used_by_playbooks: [all — via /admin_resource ingest]
intent_tags: [confluence, ingest, wiki, page, knowledge, documentation]
---

# Skill: Ingest Confluence

## Boundaries

Does NOT: update CATALOG.md (that is `collect/catalog-index`), manage supersession (that is `collect/version-supersede`), update `.source-registry.md` (that is the collector agent).

## Artifact Front-Matter Schema

Every artifact file must begin with this YAML front-matter:

```yaml
---
artifact_id: ART-YYYYMMDD-NNN
title: <descriptive topic title — not the page title>
source_type: confluence
source_url: <full Confluence page URL>
source_title: <original Confluence page title>
source_last_checked: <YYYY-MM-DD>
created: <YYYY-MM-DD>
status: current
superseded_by: null
topics: [tag1, tag2, tag3]
must_read: false
---
```

`must_read: true` only when this artifact contains information critical for any project agent to read before making decisions (e.g. core requirements, approved decisions, architectural constraints).

## Procedure

### Step 1 — Fetch the page
Use the Confluence MCP server to fetch the page content at the provided URL. If MCP is unavailable, ask the user to paste the page content.

### Step 2 — Identify topics
Read the full page and identify how many discrete topics it covers. A topic is a coherent subject that could be understood in isolation. One page may produce 1–6 artifacts.

Good decomposition signals:
- Different sections address different business concerns
- A reader would need only one artifact to answer a specific question
- Topics don't need each other to make sense

Bad decomposition signals:
- Splitting a single coherent argument across multiple artifacts
- Creating one artifact per heading regardless of content density

### Step 3 — Write one artifact per topic

For each topic:
1. Assign the next sequential artifact_id (`ART-YYYYMMDD-NNN`, `NNN+1` for each subsequent artifact from this page)
2. Write a descriptive `title` (what is this about, not the page name)
3. Write the artifact body in clean Markdown — strip Confluence-specific markup
4. Assign 3–5 `topics` tags (lowercase, hyphenated: `data-quality`, `kpi-definition`, `access-control`)
5. Set `must_read: true` only if genuinely critical for agent decision-making

### Step 4 — Cross-reference detection
If the page references other Confluence pages that have already been ingested, note the relationship in the artifact body:
> *Related artifact: ART-YYYYMMDD-NNN — [title]*

### Step 5 — Report output
Return a list of all artifact files written with their paths and titles. Do not update CATALOG.md — that is triggered separately by the collector agent.

## File Naming
```
artifacts/confluence/ART-<YYYYMMDD>-<NNN>-<slug>.md
```
Slug: first 5 words of the title, lowercase, hyphenated. Max 40 chars.

Example: `ART-20260517-001-data-retention-policy-overview.md`

## Self-Check Checklist
- [ ] All YAML front-matter fields present and populated
- [ ] `status: current` (never write a new artifact as superseded)
- [ ] Topics are 3–5 lowercase hyphenated tags
- [ ] `must_read` is true only for genuinely critical content
- [ ] Artifact body is clean Markdown — no Confluence macro remnants
- [ ] File written to `artifacts/confluence/` (not any other folder)
