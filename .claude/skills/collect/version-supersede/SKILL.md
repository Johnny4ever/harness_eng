---
name: collect/version-supersede
description: Mark an older artifact as superseded when new content replaces it. Updates the superseded artifact's front-matter, CATALOG.md, and CHANGELOG.md to maintain a clean version chain.
inputs:
  - artifact_id of the superseded artifact
  - artifact_id of the superseding artifact (new)
  - reason for supersession (brief — one sentence)
outputs:
  - superseded artifact file (front-matter updated)
  - artifacts/CATALOG.md (status updated)
  - artifacts/CHANGELOG.md (supersession logged)
model_tier_hint: haiku
used_by_playbooks: [all — via /admin_resource ingest]
intent_tags: [supersede, version, artifact version, replace, changelog, outdated]
---

# Skill: Version Supersede

## Boundaries

Does NOT: ingest new content (that is `ingest-*` skills), delete artifacts (artifacts are never deleted — only marked superseded), create the superseding artifact (that must already exist before calling this skill).

Artifacts are immutable records of what was known at a point in time. Supersession marks them as no longer current — it does not erase them.

## When to Supersede

Trigger supersession when:
- A new version of a Confluence page has been re-ingested
- A Jira epic's scope changed materially and the artifact was re-ingested
- A stakeholder corrected or replaced a previous decision captured in an ad-hoc artifact
- An artifact was split into more specific artifacts that together replace the original

Do NOT supersede for:
- Minor wording corrections (edit the artifact in place, log in CHANGELOG as "Updated")
- Adding supplementary information (create a new artifact that cross-references the original)

## Procedure

### Step 1 — Verify both artifacts exist
Confirm that both `superseded_artifact_id` and `superseding_artifact_id` files exist in the artifact library. If either is missing, stop and report the discrepancy.

### Step 2 — Update the superseded artifact's front-matter

In the superseded artifact file, update:
```yaml
status: superseded
superseded_by: ART-YYYYMMDD-NNN   # the new artifact's ID
```

Do not change any other field. Do not alter the artifact body.

### Step 3 — Update CATALOG.md

In the CATALOG.md index table:
- Find the row for the superseded artifact
- Change its `status` column value from `current` to `superseded`

### Step 4 — Update CHANGELOG.md

Prepend a new row to CHANGELOG.md:
```
| <today> | Superseded | <old_id> | <old_title> | Superseded by <new_id>: <reason> |
```

### Step 5 — Report
Return:
```
Supersession recorded:
  - <old_id> marked superseded
  - Superseded by: <new_id>
  - Reason: <reason>
  - CATALOG.md and CHANGELOG.md updated
```

## Supersession Chain
When multiple versions exist, the chain is readable via `superseded_by` fields:
```
ART-20260101-001 (superseded → ART-20260201-005)
  → ART-20260201-005 (superseded → ART-20260517-012)
    → ART-20260517-012 (current)
```

Always follow the chain from oldest to newest to reconstruct history.

## Self-Check Checklist
- [ ] Both artifact files confirmed to exist before making any changes
- [ ] Superseded artifact has `status: superseded` and `superseded_by: <new_id>` in front-matter
- [ ] CATALOG.md status column updated to `superseded` for the old artifact
- [ ] CHANGELOG.md has new `Superseded` row with reason
- [ ] Artifact body of the superseded artifact was NOT modified
- [ ] No artifact was deleted
