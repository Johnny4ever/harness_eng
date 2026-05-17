# Versioning Protocol

Every deliverable file follows this protocol before it is overwritten. This preserves full history, makes evolution traceable, and enables rollback.

## The Three Steps (always in this order)

### Step 1 — Archive the current version

Before overwriting a file, copy it to the `versions/` subfolder of its directory:

```
<deliverable-dir>/
  <filename>.md              ← current (will be overwritten)
  versions/
    <filename>-v<N>-<YYYYMMDD>.md   ← archived copy
```

Version number `<N>` is an integer starting at 1. Increment on each archive.

Example:
```
docs/projects/sales-dash/output/02-kpi/
  02-kpi-dictionary.md
  versions/
    02-kpi-dictionary-v1-20260517.md   ← first archive
    02-kpi-dictionary-v2-20260518.md   ← second archive
```

### Step 2 — Write the new version

Overwrite the canonical file (`02-kpi-dictionary.md`) with the new content.

### Step 3 — Append to DECISIONS.md

Append one line to `docs/projects/<slug>/DECISIONS.md`:

```
| <YYYY-MM-DD> | <agent-name> | <file-path> | <what changed> | <reason> |
```

Example:
```
| 2026-05-17 | generator | 02-kpi/02-kpi-dictionary.md | Added churn_rate KPI, removed bounce_rate | Stakeholder alignment CP1 — bounce_rate out of scope |
```

## VERSION-INDEX.md (optional but recommended)

Each step output folder may maintain a `VERSION-INDEX.md` that lists all versions:

```markdown
# Version Index: KPI Dictionary

| Version | Date | Author | Summary |
|---|---|---|---|
| v1 | 2026-05-17 | generator | Initial draft — 4 KPIs |
| v2 | 2026-05-18 | generator | Added churn_rate, removed bounce_rate after alignment |
```

Create this file on the second version (when the first archive is made). Update on each subsequent version.

## What Triggers the Protocol

| Trigger | Must archive? |
|---|---|
| Generator reworks a deliverable after evaluator FAIL | ✅ Yes |
| Generator updates a deliverable after user feedback at human gate | ✅ Yes |
| Planner updates `strategy.md` | ✅ Yes |
| Planner updates `project-journal.md` | ✅ Yes |
| Evaluator updates a sprint contract during iteration | ✅ Yes — update front-matter `iteration:` field and archive |
| First time a file is written (no prior version exists) | ❌ No — nothing to archive |
| Evaluator writes `eval-verdict-*` or `eval-feedback-*` files | ❌ No — these are iteration logs, not versioned deliverables |
| Collector writes artifact files under `artifacts/` | ❌ No — artifacts use supersession chains instead (see `artifacts/CHANGELOG.md`) |

## Material vs. Non-Material Changes

Only **material** changes require archiving. A material change is one that alters:
- The substance of a KPI definition, data mapping, or model design
- Scope (adds or removes deliverables, KPIs, tables)
- A decision that affects downstream steps

Non-material changes (typo fixes, formatting, adding a clarifying sentence without changing meaning) may be committed without archiving. Use judgment — when in doubt, archive.

## Rollback

To roll back a deliverable:
1. Copy the desired version from `versions/` back to the canonical path
2. Append a DECISIONS.md entry noting the rollback and reason
3. Do NOT delete intermediate versions — preserve full history

## Artifact Versioning (Collector)

Artifacts under `artifacts/` follow a different protocol — supersession chains, not file-level archiving. When a source document is updated:
- The old artifact is marked `status: superseded` in its front-matter
- A new artifact is created with a new `artifact_id`
- `artifacts/CHANGELOG.md` records the supersession link

See `artifacts/.source-registry.md` and `artifacts/CHANGELOG.md` for details.
