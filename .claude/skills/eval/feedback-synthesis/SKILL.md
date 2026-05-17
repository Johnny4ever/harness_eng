---
name: eval/feedback-synthesis
description: Convert a FAIL verdict into a specific, actionable rework brief for the generator. No praise, no softening.
inputs:
  - eval-verdict-<cp>-iter<N>.md (already in context from grading)
  - sprint-contract-<cp>.md (already in context from grading)
outputs:
  - eval-feedback-<cp>-iter<N>.md
model_tier_hint: sonnet
used_by_playbooks: [all]
intent_tags: [feedback, rework, fail, correction, iteration]
---

# Skill: Feedback Synthesis

## When to Use

Load this skill immediately after a FAIL verdict. The verdict and sprint contract are already in context — do not re-read them. This skill is typically invoked in the same evaluator session as output-grading.

## Disposition

This is a correction document, not a progress report. The generator already knows what passed — do not repeat it. Focus entirely on what failed and exactly how to fix it.

## Inputs

Already in context from output-grading (no additional file reads required):
- FAIL verdict with criterion results
- Sprint contract with criterion definitions

## Procedure

### Step 1 — Identify failing criteria

From the verdict's `failing_criteria` list, gather all FAIL criterion IDs and their evidence notes.

### Step 2 — Write one rework block per failing criterion

For each failing criterion:

1. **State the criterion ID and name** — so generator can cross-reference the contract
2. **State what PASS requires** — quote the contract's pass condition verbatim
3. **State what was found** — be specific: quote the gap, note the absence, name the missing item
4. **State the action** — one or two sentences: exactly what to add, change, or remove

**Specificity rules:**
- Name specific entities: "KPI `churn_rate`" not "one of the KPIs"
- Name specific locations: "section 3, Source Mapping table" not "in the document"
- Name specific values: "grain must be `user_id × date`" not "grain must be specified"
- Avoid vague verbs: "add", "include", "specify" over "improve", "enhance", "address"

### Step 3 — Write the "do not change" section

List criteria that passed, with one word of confirmation each. This prevents the generator from accidentally breaking passing content during rework.

### Step 4 — Add iteration budget warning

Calculate remaining iterations: `max_iterations - current_iteration`.
- If 2 remain: "2 iterations remaining before escalation."
- If 1 remains: "⚠️ FINAL ITERATION. Escalation follows if this fails."

### Step 5 — Output the file

Write to: `docs/projects/<slug>/output/eval-feedback-<cp>-iter<N>.md`

```markdown
---
checkpoint: <cp-id>
iteration: <N>
verdict: FAIL
failing_criteria: [C2, C4]
iterations_remaining: <N>
---

# Rework Brief: <Checkpoint Name> — Iteration <N>

## What to Fix

### C2 — <Criterion Name>
**Required:** <verbatim pass condition from contract>
**Found:** <specific gap — quote or precise description of what's missing>
**Action:** <specific instruction — what to write, where, with what content>

### C4 — <Criterion Name>
**Required:** <verbatim pass condition from contract>
**Found:** <specific gap>
**Action:** <specific instruction>

## Do Not Change

These criteria passed — do not alter the content satisfying them:
- C1 — <name>: ✅
- C3 — <name>: ✅

## Iteration Budget

Iteration <N> of <max_iterations>. **<iterations_remaining> iteration(s) remaining** before escalation.
[⚠️ FINAL ITERATION warning if applicable]
```

## Quality Check

- [ ] Every failing criterion has its own block
- [ ] Every block has a specific action (not just "fix this")
- [ ] No passing criteria are mentioned except in the "do not change" list
- [ ] No praise, no encouragement, no qualifiers like "great work on X but..."
- [ ] Iteration budget is correctly calculated and stated

## Lessons Learned

<!--
Append one bullet per lesson. Newest at the top. See `rules/lessons-learned-protocol.md`.
Format:
- **<YYYY-MM-DD> — <short title>.** Trigger: <project + failure pattern>. Change: <what changed>. learning_id: L-NNN
-->
