# Harness Loop

The universal pipeline every playbook runs through. No agent or skill may bypass this loop.

## Loop Shape

```
1. collector   — ingest source material into artifacts/ (optional, may pre-exist)
2. planner     — read artifacts, select playbook, produce strategy.md + journal
3. For each checkpoint defined in the playbook:
   a. evaluator  → loads eval/contract-definition skill
                 → writes sprint-contract-<cp>.md
   b. generator  → reads playbook steps for this checkpoint
                 → reads sprint-contract-<cp>.md
                 → loads each step's skill from SKILLS-CATALOG.md
                 → produces deliverables
                 → self-checks against contract checklist before signalling done
   c. evaluator  → loads eval/output-grading skill
                 → reads deliverables + sprint contract
                 → writes eval-verdict-<cp>-iter<N>.md (PASS or FAIL, never partial)
   d. On FAIL and iteration_count < max_iterations:
                 → evaluator loads eval/feedback-synthesis skill
                 → writes eval-feedback-<cp>-iter<N>.md
                 → iteration_count += 1, return to (b)
   e. On FAIL and iteration_count == max_iterations:
                 → ESCALATE: present full verdict + feedback history to user
                 → STOP — do not advance until user intervenes
   f. On PASS:
                 → present deliverables to user (human gate)
                 → WAIT for explicit user approval
                 → on approval: advance to next checkpoint, reset iteration_count to 0
4. After all checkpoints PASS and are user-approved:
   evaluator    → loads eval/retrospective skill
                → writes lesson-learned artifact to artifacts/ad-hoc/
```

## Invariants — Never Break These

| Rule | Who it constrains |
|---|---|
| Generator never advances past a checkpoint without evaluator PASS + user approval | generator |
| Evaluator verdict is PASS or FAIL only — no partial credit | evaluator |
| Planner never executes deliverable work — it only plans and hands off | planner |
| Collector never generates content — it only ingests and indexes source material | collector |
| Context resets between checkpoints — agent starts fresh with handoff block, not accumulated history | all agents |
| `max_iterations` default is 5; hard system cap is 10 — never exceed | eval-cycle |
| Any skill commit must update SKILLS-CATALOG.md in the same commit | maintainer |

## Iteration Budget

Default `max_iterations` per checkpoint: **5**

Overridable per playbook via front-matter:
```yaml
checkpoints:
  - id: cp1
    max_iterations: 3   # tighter budget for fast-moving requirements step
```

At iteration 3/5 the evaluator includes a warning in the feedback: "2 iterations remaining before escalation."

## Context Reset Protocol

At every checkpoint boundary:
1. The agent reads the prior checkpoint's `eval-verdict-<cp>.md` (pass record)
2. The agent reads the handoff block (`<!-- HANDOFF ... -->`) from the last deliverable
3. The agent reads `strategy.md` routing section only (not full strategy)
4. The agent does NOT carry forward conversation history from the prior checkpoint

This prevents "context anxiety" (per Anthropic harness article) where accumulated context degrades judgment.

## Escalation Protocol

When `max_iterations` is reached without PASS:
1. Evaluator writes `eval-escalation-<cp>.md` summarising all iterations, all verdicts, all feedback
2. Generator is paused — no further attempts
3. User is presented with: iteration count, criteria that failed on every attempt, options:
   - Relax a specific criterion (user adjusts sprint contract)
   - Provide additional context/data
   - Skip checkpoint with explicit acknowledgement of known gap
   - Abort delivery
4. Only user action unpauses the loop

## Playbook Selection (Planner's Job)

The planner selects a playbook by:
1. Reading the user's intent from the `/project` invocation
2. Matching intent keywords against playbook `description` and `triggers` fields
3. If ambiguous: presenting the top 2–3 options and asking user to choose
4. If user provides `--playbook <name>` override: use that playbook unconditionally

## Human Gate Protocol

At each checkpoint PASS, the generator presents:
- A one-line summary of what was produced
- The deliverable file path(s)
- The evaluator's PASS verdict with per-criterion confirmation
- The planned next checkpoint and what it will produce

The user must respond with an explicit approval signal ("yes", "approve", "looks good", "proceed") or a redirect. Ambiguous responses are treated as "redirect — ask for clarification."
