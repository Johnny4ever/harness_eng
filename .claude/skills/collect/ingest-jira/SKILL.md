---
name: collect/ingest-jira
description: Fetch Jira issues (epics, stories, tickets) via MCP and decompose them into discrete, topic-based artifact files with structured YAML front-matter.
inputs:
  - Jira issue URL or JQL query
  - next artifact_id sequence number (from collector agent)
  - source_type: jira
outputs:
  - artifacts/jira/ART-<date>-<NNN>-<slug>.md (one per logical topic)
model_tier_hint: sonnet
used_by_playbooks: [all — via /admin_resource ingest]
intent_tags: [jira, ingest, ticket, issue, epic, story, requirements, backlog]
---

# Skill: Ingest Jira

## Boundaries

Does NOT: update CATALOG.md (that is `collect/catalog-index`), manage supersession (that is `collect/version-supersede`), update `.source-registry.md` (that is the collector agent).

## Artifact Front-Matter Schema

Every artifact file must begin with this YAML front-matter:

```yaml
---
artifact_id: ART-YYYYMMDD-NNN
title: <descriptive topic title — not the issue summary>
source_type: jira
source_url: <full Jira issue URL or search URL>
source_title: <original issue summary or epic name>
source_last_checked: <YYYY-MM-DD>
created: <YYYY-MM-DD>
status: current
superseded_by: null
topics: [tag1, tag2, tag3]
must_read: false
jira_key: <PROJ-NNN or list of keys>
jira_type: epic | story | task | bug | spike
---
```

`must_read: true` only when this artifact contains information critical for any project agent (e.g. core acceptance criteria, approved architecture decisions, blocking constraints).

## Grouping Strategy

Jira ingestion differs from Confluence: one artifact may cover multiple related issues. Group by coherent topic, not one-issue-per-artifact.

Good grouping signals:
- An epic + its child stories → one artifact (if the epic is small)
- A cluster of bug reports about the same component → one artifact
- A spike and its output story → one artifact

Bad grouping signals:
- Putting unrelated tickets in one artifact because they share a label
- One artifact per ticket regardless of volume (produces noise)
- Mixing epics from different business domains

Typical range: 1 artifact per epic, or 1 artifact per 3–8 related stories.

## Procedure

### Step 1 — Fetch the issue(s)
Use the Jira MCP server to fetch content at the provided URL or run the provided JQL query. If MCP is unavailable, ask the user to paste the issue details.

For JQL queries: fetch up to 20 issues. If more, ask the user to narrow the query.

### Step 2 — Identify logical topics
Review all fetched issues. Group them into logical topics. Each topic becomes one artifact.

### Step 3 — Write one artifact per topic

For each topic:
1. Assign the next sequential artifact_id (`ART-YYYYMMDD-NNN`, increment for each)
2. Write a descriptive `title` (what is this about — not the ticket summary)
3. Write the artifact body in clean Markdown using this structure:

```markdown
## Summary
<1–3 sentence plain-English description of what this group of issues is about>

## Issues Covered
| Key | Type | Status | Summary |
|---|---|---|---|
| PROJ-001 | Epic | In Progress | ... |

## Requirements / Acceptance Criteria
<extracted from issue descriptions — bullet points, numbered lists>

## Key Decisions and Constraints
<any decision fields, comments marked as decisions, or constraint notes>

## Open Questions
<unresolved questions from issue comments or description>
```

4. Set `jira_key` to the primary key (or comma-separated list if multi-issue)
5. Assign 3–5 `topics` tags
6. Set `must_read: true` only if the issue defines core acceptance criteria

### Step 4 — Cross-reference detection
If any issue references Confluence pages or other Jira epics already ingested, note the relationship:
> *Related artifact: ART-YYYYMMDD-NNN — [title]*

### Step 5 — Report output
Return a list of all artifact files written with their paths and titles. Do not update CATALOG.md.

## File Naming
```
artifacts/jira/ART-<YYYYMMDD>-<NNN>-<slug>.md
```
Slug: first 5 words of the title, lowercase, hyphenated. Max 40 chars.

Example: `ART-20260517-003-checkout-flow-requirements-epic.md`

## Self-Check Checklist
- [ ] All YAML front-matter fields present including `jira_key` and `jira_type`
- [ ] `status: current` on all new artifacts
- [ ] Topics are 3–5 lowercase hyphenated tags
- [ ] Issues Covered table present with status column
- [ ] `must_read` is true only for artifacts with core acceptance criteria
- [ ] File written to `artifacts/jira/` (not any other folder)
