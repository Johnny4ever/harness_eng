# Sprint Contract Schema

A sprint contract is written by the evaluator **before** each checkpoint begins. It defines what "done" means in gradable, binary terms so the generator knows exactly what to produce and the grader knows exactly what to assess.

## File Naming

```
docs/projects/<slug>/output/sprint-contract-<checkpoint-id>.md
```

Example: `docs/projects/sales-dash/output/sprint-contract-cp2.md`

## Schema

```markdown
---
checkpoint: <cp-id>
checkpoint_name: <human-readable name>
playbook: <playbook-name>
project_slug: <slug>
iteration: 1
max_iterations: 5
created_by: evaluator
---

# Sprint Contract: <checkpoint_name>

## Deliverables

The generator MUST produce all of the following files before signalling done:

| File path | Description |
|---|---|
| `<path>` | `<what this file contains>` |

## Acceptance Criteria

Each criterion is graded PASS or FAIL. Any single FAIL → overall checkpoint FAIL.

| ID | Criterion | PASS means | FAIL means |
|---|---|---|---|
| C1 | <short name> | <concrete, observable pass condition> | <concrete, observable fail condition> |
| C2 | ... | ... | ... |

## Self-Check Checklist (generator runs before handoff)

The generator must confirm every item before writing the handoff block:

- [ ] All deliverables in the table above are written to their specified paths
- [ ] C1: <restate pass condition in imperative form>
- [ ] C2: <restate pass condition in imperative form>
- [ ] Handoff block is present at end of primary deliverable

## Grader Instructions

Guidance specific to the evaluator for this checkpoint:

- <instruction — e.g. "Be skeptical of self-reported completeness — verify file paths exist">
- <instruction — e.g. "C3 requires ALL KPIs covered — partial coverage is FAIL, not partial">
- <instruction — e.g. "If a gap is documented, that is not a FAIL — undocumented gaps are">

## Out of Scope for This Checkpoint

The evaluator must NOT penalise for missing items that belong to a later checkpoint:

- <item>
- <item>
```

## Writing Good Acceptance Criteria

The most common failure mode is vague criteria that the evaluator can rubber-stamp. Follow these rules:

**Make criteria observable, not judgmental:**

| ❌ Vague | ✅ Observable |
|---|---|
| "Discovery is thorough" | "All KPIs in the KPI dictionary have a mapped source table or a documented gap reason" |
| "Good data quality assessment" | "Each source table has null rate, row count, and freshness date recorded" |
| "Reasonable SQL" | "All SQL models pass `dbt compile` without errors" |
| "Readable wireframe" | "Every KPI in the KPI dictionary appears in at least one wireframe panel" |

**Hard threshold, no credit for partial completion:**

The evaluator applies PASS/FAIL per criterion — never "mostly done" or "good enough." If the criterion says "all KPIs mapped" and 4 of 5 are mapped, that is FAIL with specific feedback: "KPI `churn_rate` has no source mapping."

**Scope criteria to this checkpoint only:**

Do not include criteria for work that belongs to a later checkpoint. If data quality profiling is Checkpoint 2 and model design is Checkpoint 3, the Checkpoint 2 contract says nothing about the model.

## Criterion Count Guidelines

| Checkpoint type | Recommended criteria count |
|---|---|
| Requirements / alignment | 3–5 |
| Discovery / profiling | 4–7 |
| Model / SQL build | 4–6 |
| Wireframe / design | 3–5 |
| Build / QA | 5–8 |
| Release | 3–5 |

More than 8 criteria per checkpoint is a signal the checkpoint is too large — split it.

## Iteration Tracking

Each rework iteration creates a new verdict file but reuses the same contract file. The evaluator updates the `iteration:` field in the front-matter before each grading pass.

When writing feedback after a FAIL, the evaluator must reference criterion IDs (C1, C2, ...) so the generator knows exactly which criteria to address without re-reading the full contract.
