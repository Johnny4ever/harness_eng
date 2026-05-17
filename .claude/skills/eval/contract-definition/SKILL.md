---
name: eval/contract-definition
description: Define sprint contracts with observable, binary acceptance criteria before a checkpoint begins
inputs:
  - strategy.md (step section for the target checkpoint)
  - playbook checkpoint definition
  - planner handoff block
outputs:
  - sprint-contract-<cp>.md
model_tier_hint: opus
used_by_playbooks: [all]
intent_tags: [contract, criteria, acceptance, before checkpoint, define done]
---

# Skill: Contract Definition

## When to Use

Load this skill when the evaluator agent needs to write a sprint contract before a checkpoint begins. Always runs before the generator starts work for a checkpoint.

## Inputs

| Input | Source | Read limit |
|---|---|---|
| Strategy step section | `strategy.md` | Read only the rows for this checkpoint's steps |
| Playbook checkpoint block | `.claude/playbooks/<name>.md` | Read only the checkpoint definition block |
| Planner handoff | `<!-- HANDOFF -->` block in strategy.md | Embedded — no extra file read |

Total file reads: 2 (strategy + playbook).

## Procedure

### Step 1 — Extract deliverables

From the strategy step section, list every file the generator must produce for this checkpoint. Be exact: full relative path, not just the step name.

### Step 2 — Draft acceptance criteria

For each deliverable and each known requirement, write one criterion. Use the test:
> "Can I look at the deliverable and answer YES or NO to this criterion without judgment?"

If the answer requires judgment ("is it good?", "is it thorough?"), rewrite it as something observable ("does it contain X for every Y?").

**Criterion template:**

| Field | Content |
|---|---|
| ID | C1, C2, C3 … (sequential) |
| Name | 3–5 words |
| PASS means | Observable condition — present and verifiable in the deliverable |
| FAIL means | Observable condition — missing, absent, or contradicted in the deliverable |

**Common criterion patterns by domain:**

| Domain | Example criterion |
|---|---|
| Requirements | "All KPIs named in the brief appear in the requirement doc" |
| KPI definition | "Each KPI has: formula, grain, filter, owner, source hint" |
| Data discovery | "Every KPI has either a source table or a documented gap reason" |
| Semantic model | "Every fact table has grain declared; every dimension has surrogate key declared" |
| SQL / dbt | "All models compile without errors" |
| Wireframe | "Every KPI in the KPI dictionary appears in at least one panel" |
| Build | "Dashboard loads in the target platform without errors" |
| Release | "Data refresh schedule is configured and tested" |

### Step 3 — Write the self-check checklist

Restate each PASS condition as an imperative sentence the generator checks off:
- "Confirm KPI `churn_rate` has source table documented" (not "check discovery")

### Step 4 — Write grader instructions

Add 2–4 instructions specific to this checkpoint that remind the grader to be skeptical in the areas most likely to have soft gaps:
- Which criteria are most commonly failed?
- Where might the generator assert completion without evidence?
- What placeholders are unacceptable?

### Step 5 — Write out-of-scope section

List 2–4 items that belong to a later checkpoint. This prevents the grader from penalising work that isn't due yet.

### Step 6 — Output the file

Write to: `docs/projects/<slug>/output/sprint-contract-<cp>.md`

Follow the schema in `rules/sprint-contract-schema.md` exactly.

## Quality Check (before writing the file)

- [ ] Every criterion is binary — a neutral observer can grade it YES/NO
- [ ] No criterion uses words like "thorough", "good", "appropriate", "reasonable"
- [ ] Criterion count is within guidelines (3–8 per checkpoint)
- [ ] Self-check checklist matches criteria 1:1
- [ ] Out-of-scope section is non-empty (there is always something that comes later)

## Lessons Learned

<!--
Append one bullet per lesson. Newest at the top. See `rules/lessons-learned-protocol.md`.
Format:
- **<YYYY-MM-DD> — <short title>.** Trigger: <project + failure pattern>. Change: <what changed>. learning_id: L-NNN
-->
