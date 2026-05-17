---
name: collect/ingest-adhoc
description: Decompose ad-hoc pasted content (emails, Slack threads, meeting notes, PDFs, docs) into discrete topic-based artifact files with structured YAML front-matter.
inputs:
  - pasted text or file content (any format)
  - source description provided by user (e.g. "Slack thread from #data-team", "email from Jane re: KPI definitions")
  - next artifact_id sequence number (from collector agent)
  - source_type: adhoc
outputs:
  - artifacts/ad-hoc/ART-<date>-<NNN>-<slug>.md (one per topic)
model_tier_hint: sonnet
used_by_playbooks: [all — via /admin_resource ingest]
intent_tags: [adhoc, ingest, paste, email, slack, meeting notes, document, text]
---

# Skill: Ingest Ad-Hoc

## Boundaries

Does NOT: update CATALOG.md (that is `collect/catalog-index`), manage supersession (that is `collect/version-supersede`), update `.source-registry.md` (that is the collector agent). Does NOT fetch content from URLs — that is `ingest-confluence` or `ingest-jira`.

This skill handles any content the user pastes directly into the conversation, or any file content that doesn't fit a structured MCP source.

## Artifact Front-Matter Schema

```yaml
---
artifact_id: ART-YYYYMMDD-NNN
title: <descriptive topic title>
source_type: adhoc
source_url: null
source_title: <user-provided description of the source>
source_last_checked: <YYYY-MM-DD>
created: <YYYY-MM-DD>
status: current
superseded_by: null
topics: [tag1, tag2, tag3]
must_read: false
adhoc_format: email | slack | meeting-notes | document | pdf | other
---
```

`must_read: true` only when this artifact contains information critical for any project agent (e.g. stakeholder-approved decisions, architectural constraints from leadership).

## Procedure

### Step 1 — Confirm source description
If the user has not described the source (who wrote it, what channel/meeting/document it came from, approximate date), ask before proceeding:
> "Before I ingest this, can you briefly describe the source? (e.g. 'Email from Sarah re: data requirements, 2026-05-10' or 'Slack thread #analytics-team')"

If the user says "just ingest it", use `source_title: "User-provided content — no source description"`.

### Step 2 — Detect format
Identify the content format:
- **Email**: has To/From/Subject headers or clearly narrative prose directed to a person
- **Slack**: threaded messages, @mentions, emoji reactions, short paragraphs
- **Meeting notes**: agenda items, action items, attendees, timestamps
- **Document / spec**: structured headings, formal prose
- **PDF** (content pasted): citation-style, academic or formal structure
- **Other**: anything that doesn't fit above

Set `adhoc_format` accordingly.

### Step 3 — Identify topics
Read the full content and identify discrete topics (as in `ingest-confluence`). Most ad-hoc content = 1–3 artifacts.

For meeting notes: separate "decisions made" from "action items" from "background discussion" if they are substantively different.

### Step 4 — Write one artifact per topic

For each topic:
1. Assign the next sequential artifact_id
2. Write a descriptive title
3. Write a clean Markdown body. Extract signal, remove conversational noise:
   - Strip email signatures, Slack reaction counts, "thanks!" filler
   - Preserve: decisions, requirements, constraints, open questions, action items, named owners
4. Use this structure where appropriate:

```markdown
## Context
<1–2 sentences: who, when, why this content exists>

## Key Points
<bulleted extraction of the most important information>

## Decisions Made
<only if the source contains explicit decisions — bullet list>

## Action Items
<only if the source contains action items — with owner if named>
| Action | Owner | Due |
|---|---|---|

## Open Questions
<unresolved questions from the content>
```

5. Assign 3–5 `topics` tags
6. Set `must_read: true` if a decision/constraint from leadership is present

### Step 5 — Report output
Return a list of all artifact files written with their paths and titles. Do not update CATALOG.md.

## File Naming
```
artifacts/ad-hoc/ART-<YYYYMMDD>-<NNN>-<slug>.md
```
Slug: first 5 words of the title, lowercase, hyphenated. Max 40 chars.

Example: `ART-20260517-005-stakeholder-kpi-decisions-email.md`

## Self-Check Checklist
- [ ] Source description confirmed with user (or flagged as undescribed)
- [ ] `adhoc_format` field populated
- [ ] All YAML front-matter fields present
- [ ] `status: current` on all new artifacts
- [ ] Conversational noise stripped; signal preserved
- [ ] File written to `artifacts/ad-hoc/` (not any other folder)

## Lessons Learned

<!--
Append one bullet per lesson. Newest at the top. See `rules/lessons-learned-protocol.md`.
Format:
- **<YYYY-MM-DD> — <short title>.** Trigger: <project + failure pattern>. Change: <what changed>. learning_id: L-NNN
-->
