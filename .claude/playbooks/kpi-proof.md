---
name: kpi-proof
description: >
  Minimal proof-of-concept playbook. Single checkpoint, single step.
  Takes a requirement document or brief as input and produces a KPI dictionary.
  Used to validate the generic generator + skill + evaluator loop end-to-end
  before committing to full BI playbook migration.
triggers: [kpi proof, test harness, validate loop, kpi only]
output_root: docs/projects/<slug>/output/
max_iterations_default: 5
---

# Playbook: KPI Proof

## Purpose

This playbook exists solely to verify the harness loop works end-to-end:
1. Evaluator writes a sprint contract
2. Generator loads `bi/kpi-definition` skill and produces a KPI dictionary
3. Evaluator grades with hard thresholds
4. Rework loop fires on FAIL
5. Human gate fires on PASS

Do not use this playbook for real deliveries. Use `bi-dashboard` instead.

## Input Requirements

Before running this playbook, the user must provide one of:
- A structured requirement document at `docs/projects/<slug>/output/01-requirement/01-requirement.md`
- A raw brief as direct context in the `/run` invocation (generator creates a minimal requirement summary as step 0)

## Steps

```yaml
steps:
  - id: 02-kpi
    skill: bi/kpi-definition
    output: 02-kpi/02-kpi-dictionary.md
    depends_on: []        # requirement doc is pre-existing input, not a step
    parallel_with: []
```

## Checkpoints

```yaml
checkpoints:
  - id: cp1
    name: KPI Dictionary
    after_steps: [02-kpi]
    human_gate: true
    max_iterations: 5
    eval_contract_hints:
      - "Every KPI from the input brief must appear in the dictionary"
      - "Every KPI must have all 11 fields populated — no silent blanks"
      - "Every TBD must reference a numbered open question in the ambiguity log"
      - "Date logic must be explicit for every KPI — no implicit periods"
      - "Source candidates must be labelled unconfirmed"
      - "Ambiguity log must exist and be non-empty"
```

## Output Structure

```
docs/projects/<slug>/output/
  STATUS.md
  sprint-contract-cp1.md
  eval-verdict-cp1-iter<N>.md
  eval-feedback-cp1-iter<N>.md    (only if FAIL)
  02-kpi/
    02-kpi-dictionary.md
    VERSION-INDEX.md              (from second version onward)
    versions/                     (archived prior versions)
```

## Success Criteria for This Proof

The harness loop is proven when ALL of the following are observed:
1. Evaluator produces `sprint-contract-cp1.md` with binary, observable criteria
2. Generator reads `SKILLS-CATALOG.md`, resolves `bi/kpi-definition`, loads the skill
3. Generator produces `02-kpi-dictionary.md` with all 11 fields per KPI
4. Generator writes a valid `<!-- HANDOFF -->` block
5. Evaluator grades against the contract — not a vague impression
6. If any criterion FAILs: feedback brief is specific (names KPIs, fields, locations)
7. If rework occurs: versioning protocol fires before overwrite
8. Human gate appears after PASS before any further action

## Known Limitations

- This playbook does not test parallelisation (single step)
- This playbook does not test multi-checkpoint sequencing
- This playbook does not test cross-cutting skills (source-enablement, governance-check)
- These are tested in `bi-dashboard` playbook (Phase 4)
