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

### Step 7 — Append to SKILL.metrics.md (PASS or ESCALATED only)

Skip this step on a FAIL that is not yet at the iteration cap — only the final outcome of a checkpoint is recorded.

For each skill listed in the sprint contract's Deliverables table that produced output for this checkpoint:

1. Open `.claude/skills/<category>/<skill>/SKILL.metrics.md`. If the front-matter exists but the log is empty (the Phase 8.1 placeholder), populate the first row. If the file does not exist at all, create it with the front-matter from `rules/skill-metrics-protocol.md`.

2. Append one row to the `## Per-Invocation Log` table:

| Column | Value |
|---|---|
| Date | Today (YYYY-MM-DD) |
| Project | Project slug from STATUS.md |
| Checkpoint | The checkpoint id (e.g. `cp1`) |
| Verdict | `PASS` or `ESCALATED` |
| Iter to PASS | Iteration number that finally passed; `—` on ESCALATED |
| Failed criteria | Arrow-separated history (e.g. `C2,C3 → C2 → —` means iter 1 failed C2+C3, iter 2 failed C2, iter 3 passed). `—` if first-pass PASS |
| Notes | 1-line summary; cite active `learning_id` if applicable |

3. Recompute and update the front-matter aggregates:
   - `total_invocations` = count of log rows
   - `first_pass_pass_count` = rows where Iter to PASS = 1
   - `first_pass_pass_rate` = first_pass_pass_count / total_invocations (2 decimal places)
   - `avg_iterations_to_pass` = mean of "Iter to PASS" across PASS rows only (1 decimal place)
   - `escalation_count` = rows where Verdict = ESCALATED
   - `last_updated` = today

4. If the log now exceeds 200 rows, perform rollover per `rules/skill-metrics-protocol.md`.

This append is structured and atomic. Do not edit the rest of the file. Do not delete any rows.

## Anti-Patterns to Avoid

| Anti-pattern | What to do instead |
|---|---|
| "This is mostly good, just missing X" | Grade it FAIL on criterion CX — feedback will say exactly what's missing |
| "The generator explained in chat that Y is covered" | Re-read the deliverable file. If Y is not in the file, it is not covered. |
| "I'll give partial credit since 4 of 5 KPIs are mapped" | FAIL. The criterion says all KPIs. Feedback: "KPI `churn_rate` has no source mapping." |
| "The criterion is vague so I'll interpret generously" | Apply the stricter interpretation. Flag the vague criterion in the retrospective. |
| Praising what passed before noting what failed | Lead with the overall verdict. Evidence for all criteria. No softening. |

## Lessons Learned

<!--
Append one bullet per lesson. Newest at the top. See `rules/lessons-learned-protocol.md`.
Format:
- **<YYYY-MM-DD> — <short title>.** Trigger: <project + failure pattern>. Change: <what changed>. learning_id: L-NNN
-->
