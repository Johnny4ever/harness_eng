---
name: eval/output-grading
description: Grade generator deliverables PASS/FAIL against sprint contract with hard thresholds. Skeptical by default.
inputs:
  - sprint-contract-<cp>.md
  - primary deliverable(s)
  - prior verdict (rework iterations only)
outputs:
  - eval-verdict-<cp>-iter<N>.md
model_tier_hint: opus
used_by_playbooks: [all]
intent_tags: [grade, evaluate, verdict, assess, quality check, pass fail]
---

# Skill: Output Grading

## When to Use

Load this skill when the evaluator agent needs to grade a generator deliverable after the generator signals done.

## Disposition

You are grading, not reviewing. Your default is FAIL. Issue PASS only when every criterion is clearly and demonstrably met with evidence you can point to in the deliverable.

## Inputs

| Input | Source | Read limit |
|---|---|---|
| Sprint contract | `sprint-contract-<cp>.md` | Full read |
| Primary deliverable | Path from generator handoff `must_reads` | Full read |
| Prior verdict | `eval-verdict-<cp>-iter<N-1>.md` | Rework only — replaces one must_read slot |

Total file reads: 2–3.

## Procedure

### Step 1 — Read the sprint contract completely

Do not begin grading until you have read every criterion. Understand the pass threshold for each criterion before looking at the deliverable.

### Step 2 — Read the deliverable

Read the deliverable from top to bottom. Do not grade as you read — form a complete picture first.

### Step 3 — Grade each criterion

For each criterion ID (C1, C2, ...):

1. State what PASS requires (quote from contract)
2. Find the specific content in the deliverable that addresses it
3. Apply the hard-threshold test: does the content clearly satisfy the criterion?
4. Record your evidence — a quote, a line reference, or an explicit note of absence

**Hard threshold rules:**

| Situation | Grade |
|---|---|
| Criterion clearly satisfied with evidence | ✅ PASS |
| Criterion partially satisfied (some items present, some missing) | ❌ FAIL |
| Criterion satisfied by assertion without evidence | ❌ FAIL |
| Criterion addressed with a placeholder ("TBD", "to be confirmed") | ❌ FAIL |
| Criterion addressed with a reference to an appendix that doesn't exist | ❌ FAIL |
| Criterion is ambiguous in the contract | Apply stricter interpretation → FAIL if unclear |
| Generator's explanation in chat addresses the criterion | ❌ FAIL — grade the file, not the chat |

### Step 4 — Determine overall verdict

- Any criterion FAIL → Overall: **FAIL**
- All criteria PASS → Overall: **PASS**

No exceptions. No "close enough." No weighted scoring.

### Step 5 — Write the verdict file

Write to: `docs/projects/<slug>/output/eval-verdict-<cp>-iter<N>.md`

```markdown
---
checkpoint: <cp-id>
iteration: <N>
verdict: PASS | FAIL
failing_criteria: []   # list criterion IDs that failed, empty if PASS
---

# Verdict: <Checkpoint Name> — Iteration <N>

## Overall: ✅ PASS | ❌ FAIL

## Criterion Results

| ID | Criterion | Result | Evidence |
|---|---|---|---|
| C1 | <name> | ✅ PASS | "<quoted content>" or "Section X contains Y" |
| C2 | <name> | ❌ FAIL | "Required: <X>. Found: <what was actually there or absent>" |

## Summary

<PASS: "All N criteria met. Checkpoint <cp> is approved for human review.">
<FAIL: "Criteria C2, C4 failed. See feedback brief for rework instructions.">
```

### Step 6 — Update STATUS.md

- PASS: mark the step as `✅ COMPLETE` with checkpoint and iteration
- FAIL: mark the step as `🔄 IN PROGRESS` (rework pending), update iteration count

## Anti-Patterns to Avoid

| Anti-pattern | What to do instead |
|---|---|
| "This is mostly good, just missing X" | Grade it FAIL on criterion CX — feedback will say exactly what's missing |
| "The generator explained in chat that Y is covered" | Re-read the deliverable file. If Y is not in the file, it is not covered. |
| "I'll give partial credit since 4 of 5 KPIs are mapped" | FAIL. The criterion says all KPIs. Feedback: "KPI `churn_rate` has no source mapping." |
| "The criterion is vague so I'll interpret generously" | Apply the stricter interpretation. Flag the vague criterion in the retrospective. |
| Praising what passed before noting what failed | Lead with the overall verdict. Evidence for all criteria. No softening. |
