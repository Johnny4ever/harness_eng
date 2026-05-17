---
lesson_id: L-001
title: TBD as sole field content is never acceptable
created: 2026-05-17
applies_to: [bi/kpi-definition, generic/requirement-intake, data/discovery, all skills with field-content deliverables]
status: active
superseded_by: null
trigger_projects: [kpi-proof-dryrun]
---

# L-001: TBD as sole field content

## The trap

Self-check items that validate "TBD is present *and* references an open question" instead of forbidding TBD entirely in field values. The generator then interprets `"TBD — see OQ-2"` as an acceptable value, producing deliverables where critical fields contain no real content. The evaluator FAILs the checkpoint because the deliverable does not actually answer the question — but by then the iteration budget has been consumed.

## The rule

A field's content must be a substantive best-available value, even if uncertain. Uncertainty is recorded in an Open Questions section adjacent to the field — never inside the field itself.

If a value is genuinely unknowable at this stage, the deliverable must:
1. Provide the best-available approximation with explicit assumptions
2. Reference the open question by ID in an Open Questions section
3. Specify what evidence would resolve the uncertainty

`"TBD"`, `"TBC"`, `"unknown"`, `"see OQ-N"` as the entire content of a field is forbidden.

## How to reference

In a skill's self-check checklist:
```markdown
- [ ] No field contains TBD as its only content (see lessons/L-001)
```

In a skill's step instructions:
```markdown
### Step N — Write field values
For each field, write a best-available substantive value. If uncertain, write
the value with explicit assumptions and add an Open Questions entry pointing
back. Never use "TBD" as the entire field content (see lessons/L-001).
```

## History

- **2026-05-17**: Recorded. Source: Phase 3 dry-run of the harness loop. Trigger was kpi-proof-dryrun project where `date_handling: "TBD"` for `trial_to_paid_conversion_rate` caused evaluator FAIL on C2/C3. Skill patched in commit `0de4dd1` ("fix(skill/bi-kpi-definition): TBD+OQ is not a substitute for field content"). Generalised into this lesson during Phase 8.0.
