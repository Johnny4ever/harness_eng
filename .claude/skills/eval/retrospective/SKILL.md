---
name: eval/retrospective
description: Post-delivery retrospective that captures iteration patterns, skill performance, and playbook improvement recommendations.
inputs:
  - STATUS.md
  - eval-verdict-* files for this delivery (up to 3)
outputs:
  - artifacts/ad-hoc/ART-<date>-<NNN>-retro-<slug>.md
model_tier_hint: sonnet
used_by_playbooks: [all]
intent_tags: [retrospective, lesson learned, review, post-delivery, improve]
---

# Skill: Retrospective

## When to Use

Load this skill after all checkpoints have passed and the delivery is marked complete in `STATUS.md`. This is the final evaluator action in the harness loop.

## Inputs

| Input | Source | Read limit |
|---|---|---|
| STATUS.md | `docs/projects/<slug>/output/STATUS.md` | Full read |
| Verdict files | `eval-verdict-cp*-iter*.md` files for this project | Up to 3 files |

Total file reads: 2–4.

## Procedure

### Step 1 — Analyse iteration patterns

From STATUS.md and verdict files, identify:
- Which checkpoints required more than 1 iteration?
- Which criteria failed most often across iterations?
- Which steps completed on first attempt?

### Step 2 — Identify root causes

For each multi-iteration checkpoint, determine why rework was needed. Common root causes:

| Root cause | Signal | Recommendation |
|---|---|---|
| Skill content too vague | Same criterion fails in multiple iterations with similar gaps | Improve skill procedure or add examples |
| Sprint contract criterion too broad | Grader interpretation differs from generator's | Tighten criterion wording in contract template |
| Missing source data | Criterion about completeness fails repeatedly | Add source-enablement step to playbook |
| Scope creep from generator | Generator produces content outside the checkpoint | Strengthen role-boundary section in generator agent |
| Contract criterion too strict for this domain | Escalation reached on first use of a playbook | Calibrate criterion for this playbook's typical output |

### Step 3 — Rate skill performance

For each skill used in this delivery:
- **Effective**: step completed on first iteration
- **Needs improvement**: step required rework; note what criterion failed
- **Not used** (native skill substituted): note which native skill was preferred

### Step 4 — Write recommendations

Produce 3–7 specific, actionable recommendations. Each recommendation must name:
- What to change (skill file, contract template, playbook, or agent)
- Where exactly (file path + section)
- What to add/remove/reword

Vague recommendations ("improve the discovery skill") are not acceptable. Specific ones ("add an example in `skills/data/discovery/SKILL.md` showing how to document a null-rate gap in the source map") are.

### Step 5 — Write the artifact

Write to: `artifacts/ad-hoc/ART-<YYYYMMDD>-<NNN>-retro-<slug>.md`

Follow the artifact front-matter convention used in `artifacts/CATALOG.md`.

```markdown
---
artifact_id: ART-<YYYYMMDD>-<NNN>
title: Retrospective — <project name>
source_type: ad-hoc
created: <date>
project_slug: <slug>
playbook: <name>
status: current
---

# Retrospective: <Project Name>

**Delivery date:** <date>
**Playbook used:** <name>
**Total checkpoints:** <N>
**Total iterations across all checkpoints:** <N>

## Checkpoint Summary

| Checkpoint | Steps | Iterations needed | Root cause (if > 1) |
|---|---|---|---|
| CP1 | 01, 02, 03 | 1 | — |
| CP2 | 04, 05, 06, 07 | 3 | Discovery criterion C3 too broad |
| CP3 | 08 | 2 | Wireframe missing KPI from late addition |

## Skill Performance

| Skill | Result | Notes |
|---|---|---|
| `generic/requirement-intake` | ✅ Effective | First-pass success |
| `data/discovery` | ⚠️ Needs improvement | C3 failed twice — see below |
| `anthropic-skills:docx` | ✅ Effective (native) | Used instead of local `generic/documentation` |

## Root Cause Detail

### CP2 — data/discovery — C3 repeated failure
<Analysis>

## Recommendations

### 1. Tighten C3 criterion template in `skills/data/discovery/SKILL.md`
**Current:** "All KPIs have a source mapping or documented gap"
**Problem:** "documented gap" accepted a one-word entry without grain/availability date
**Change:** Add to SKILL.md checklist: "A documented gap must include: reason, who owns the data, estimated availability date"

### 2. Add example of null-rate documentation to `skills/data/quality-profiling/SKILL.md`
...

## What Worked Well

<Patterns that should be preserved or adopted in other playbooks>
```

### Step 6 — Update CATALOG.md

Append the new artifact entry to `artifacts/CATALOG.md` using the standard catalog row format.
