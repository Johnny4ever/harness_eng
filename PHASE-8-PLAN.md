# Phase 8 — Meta-Learner: Plan

**Status:** Draft, awaiting user review and decision lock-in
**Branch:** `plan/phase-8-meta-learner`
**Depends on:** PR #2 (Phase 7 atomic switch) must land on `main` before Phase 8 starts
**Companion to:** `RESTRUCTURE-PLAN.md` (Phases 0–7, completed)

---

## TL;DR

Add a fifth agent (`meta-learner`) and a small set of supporting files that let the harness *learn from its own history* — without ever silently mutating itself. The meta-learner reads accumulated evaluation records, finds patterns of repeated failure, and writes **proposal documents** that you review and apply manually via git PRs.

The agent **never edits** `.claude/` files. It is an analyst, not an editor.

---

## Goals

| # | Goal |
|---|---|
| G1 | After N projects, surface concrete, evidence-backed proposals for skill / agent / rule improvements |
| G2 | Make institutional memory durable — every skill remembers *why* it is the way it is |
| G3 | Detect when a recent change regressed quality so we can revert before damage accumulates |
| G4 | Keep the harness deterministic: same inputs → same outputs, no hidden state drift |
| G5 | Stay within the existing context-budget discipline — meta-learner has its own hard read limit |

## Non-Goals (explicitly out of scope)

| # | Non-Goal | Why |
|---|---|---|
| NG1 | Automatic application of proposals | Defeats the audit trail; risks runaway prompt drift |
| NG2 | Continuous self-improvement loop | We want batched, deliberate review — not an always-on optimizer |
| NG3 | A numeric "skill score" leaderboard | Invites Goodhart's-Law gaming of the evaluator |
| NG4 | Cross-project deliverable scanning | Too expensive; eval verdicts + retrospectives are sufficient signal |
| NG5 | Replacing the evaluator's PASS/FAIL with graded scores | Hard thresholds are working — don't soften them to feed a metric |
| NG6 | Modifying `.claude/` files programmatically | All changes go through git PRs reviewed by a human |

---

## Design Principles

1. **Notice ≠ Change.** Recording observations and modifying configuration are separate layers, separated by a human gate.
2. **Evidence threshold.** No proposal without ≥ 3 data points (configurable, see Decision 4). One-off failures are noise.
3. **Scoped lessons.** Every lesson is classified — project-local, domain-wide, or universal. Most stay project-local; promotion to wider scope must be argued.
4. **Reject-memory.** Once you reject a proposal, the meta-learner records it and will not re-propose the same change.
5. **Regression guard.** Every accepted change is tagged with a `learning_id`; the next 3 invocations of the affected skill auto-include a "did this help?" check.
6. **The meta-learner has the same context budget discipline as every other agent** — explicit per-invocation file read cap.

---

## Architecture

```
Existing harness (post-Phase 7):
  /project → planner → [evaluator ↔ generator] → human gate → next checkpoint
                            ↓
                  writes eval-verdict, eval-feedback, retrospective
                            ↓
                  NEW: also appends to .claude/skills/<skill>/SKILL.metrics.md

New, runs outside the main loop:
  /meta-learn → meta-learner (Opus)
                  ├─ reads: SKILL.metrics.md (all skills)
                  ├─ reads: review-iter*.md (all projects)
                  ├─ reads: docs/meta/rejected/ (past rejections)
                  └─ writes: docs/meta/proposals/<date>-<target>.md

You manually:
  - Review proposal → approve → apply diff in a normal PR
  - Or reject → move file to docs/meta/rejected/
```

The meta-learner cannot write anywhere under `.claude/`. Enforced by its system prompt and by reviewer discipline.

---

## The Five Pieces

### Piece 1 — Per-skill metrics sidecar

**Path:** `.claude/skills/<category>/<skill>/SKILL.metrics.md`

Append-only telemetry. Updated by the evaluator immediately after writing each verdict.

**Schema:**
```markdown
---
skill: bi/kpi-definition
first_recorded: 2026-05-17
total_invocations: 5
first_pass_pass_count: 3
first_pass_pass_rate: 0.60
avg_iterations_to_pass: 1.4
last_updated: 2026-08-22
---

# Metrics: bi/kpi-definition

## Per-Invocation Log
| Date | Project | Checkpoint | Verdict | Iter to PASS | Failed criteria | Notes |
|---|---|---|---|---|---|---|
| 2026-05-17 | sales-dash | cp1 | PASS | 2 | C2 → C0 | TBD-as-content fixed in iter 2 |
| 2026-06-04 | churn-analysis | cp1 | PASS | 1 | — | clean first-pass |
| 2026-07-12 | finance-mrr | cp1 | PASS | 3 | C2, C3 → C2 → C0 | grain misread twice |

## Active Learning Tags
- L-007 (applied 2026-07-15): "explicit grain decision tree" — verify in next 3 invocations
```

Constraints:
- File capped at 200 rows; older rows roll to `SKILL.metrics-archive-<year>.md`
- Evaluator writes one row per checkpoint completion (PASS or final FAIL)
- Failed criteria column lists which IDs failed in iter 1, then in iter 2, etc.

### Piece 2 — `## Lessons Learned` section in SKILL.md

Every skill's `SKILL.md` gets a new section near the end:

```markdown
## Lessons Learned
- **2026-05-17 — Forbid TBD as sole field content.**
  Trigger: dry-run revealed evaluator FAIL'd C2/C3 when generator used
  "TBD — OQ-2" as a date_handling value. Self-check items must mirror
  evaluator hard-threshold rules exactly, not create a softer version.
  learning_id: L-001
- **2026-08-22 — Disambiguate event-based vs snapshot grain.**
  Trigger: 3 projects (sales-dash, churn-analysis, finance-mrr) failed C2 on
  grain interpretation when source was event-stream. Added Step 2.5 decision tree.
  learning_id: L-007
```

This is institutional memory embedded in the artifact. Anyone reading the skill sees not just what it does but why it does it that way.

### Piece 3 — Anti-pattern lesson library

**Path:** `.claude/lessons/L-<NNN>-<slug>.md`

For lessons that apply across multiple skills, distill into a referenceable note instead of inlining the clause N times:

```markdown
---
lesson_id: L-001
title: TBD as sole field content is never acceptable
created: 2026-05-17
applies_to: [all skills with field-content deliverables]
status: active
---

# L-001: TBD as sole field content

## The trap
Self-check items that validate "TBD is present with an open question reference" instead of forbidding TBD entirely. Generator interprets "TBD" as acceptable and produces empty deliverables.

## The rule
A field's content must be a substantive best-available value, even if uncertain. Mark the uncertainty in an Open Questions section — do not put "TBD" in the field itself.

## How to reference
Skills can cite this lesson by ID in their self-check checklist:
  - [ ] No field contains TBD as its only content (see lessons/L-001)
```

Benefits:
- Skills stay under the 300-line cap as lessons accumulate
- One canonical statement of the rule (DRY)
- Easy to audit which skills enforce which lessons

### Piece 4 — Proposal documents

**Path:** `docs/meta/proposals/<YYYY-MM-DD>-<target>.md`

Written by the meta-learner. Human reviews and either accepts (applies the diff in a PR) or rejects (moves the file to `docs/meta/rejected/`).

**Schema:**
```markdown
---
proposal_id: PROP-2026-08-22-001
created: 2026-08-22
target: .claude/skills/bi/kpi-definition/SKILL.md
target_type: skill | agent | rule | lesson | playbook
scope: project-local | domain | universal
status: open | accepted | rejected | superseded
evidence_count: 3
---

# Proposal: Add grain disambiguation step to bi/kpi-definition

## Evidence
Pattern observed in metrics:
- sales-dash (cp1, iter 2): C2 failed — "grain ambiguous when source is event"
- churn-analysis (cp1, iter 2): C2 failed — same root cause
- finance-mrr (cp1, iter 3): C2 + C3 failed — same root cause

Common feedback theme across all three: generator defaulted to snapshot grain
when source was event-based; evaluator demanded event-level grain.

## Hypothesis
Skill lacks explicit guidance on event-vs-snapshot grain selection. Generator
falls back to a default that often conflicts with semantic intent.

## Proposed Change
Insert new Step 2.5 between current Step 2 and Step 3 of bi/kpi-definition:

\`\`\`
### Step 2.5 — Disambiguate grain when source is event-based
If the source system is an event stream (clickstream, transaction log,
audit log), choose between event-level and aggregated grain explicitly:
- Event-level: one row per event; preserves all detail
- Aggregated: rolled to time bucket or entity; loses detail but faster
Record the choice and reason in the KPI record's grain field.
\`\`\`

Also add to self-check:
- [ ] If source is event-based, grain choice is explicit (event vs aggregate)

## Expected Outcome
- C2 first-pass FAIL rate drops from 60% to <20% within next 3 invocations
- Average iterations to PASS for cp1 drops from 2.0 to ≤1.3

## Tracking
Tag with learning_id L-007. Next 3 projects using this skill auto-include
a "did L-007 help?" line in their retrospective.

## Rollback Criterion
If after 3 tagged invocations C2 FAIL rate is unchanged or worse, propose revert.

## Risk Assessment
Low. Adds a clarifying step; does not change any existing logic. No downstream
skill depends on the absence of Step 2.5.
```

### Piece 5 — The `meta-learner` agent

**Path:** `.claude/agents/meta-learner.md`
**Model:** `claude-opus-4-7` (high-stakes reasoning, runs infrequently)
**Read budget:** 12 files per invocation (above other agents because aggregation is its job)

**Inputs (allowed reads only):**
- All `SKILL.metrics.md` files (cheap — already pre-aggregated)
- `docs/projects/*/reviews/review-iter*.md` (retrospectives)
- `docs/projects/*/project-journal.md` (project context, routing section only)
- `docs/meta/rejected/*.md` (past rejections, to avoid repeating)
- `.claude/lessons/*.md` (existing lesson library)

**Forbidden reads:** raw deliverable files, `eval-verdict-*`, `eval-feedback-*` (these are too numerous; signal lives in metrics + retrospectives).

**Forbidden writes:** anything under `.claude/`. Can only write to `docs/meta/proposals/`.

**Procedure:**
1. Aggregate metrics across all skills — flag any with first-pass FAIL rate ≥ 50% and ≥ 3 invocations
2. Aggregate retrospective themes — flag any pattern appearing ≥ 3 times
3. For each flag, read past proposals + rejections to check if already addressed
4. Draft one proposal per genuine pattern (skip if duplicate of pending or rejected)
5. Write proposals to `docs/meta/proposals/`
6. Print summary: N proposals drafted, M patterns skipped (already-known)

---

## File-by-File Deliverables

### New files
| Path | Content |
|---|---|
| `.claude/agents/meta-learner.md` | Agent prompt (Opus, read-only over `.claude/`) |
| `.claude/commands/meta-learn.md` | Slash command routing to meta-learner |
| `.claude/skills/eval/metrics-recorder/SKILL.md` | (Optional, see Decision 8) Sub-skill loaded by evaluator after grading |
| `.claude/lessons/README.md` | Anti-pattern library overview, how to reference, how to add |
| `.claude/lessons/L-001-tbd-as-sole-content.md` | Seed lesson — distilled from the TBD bug |
| `.claude/rules/lessons-learned-protocol.md` | How to write the `## Lessons Learned` section in skills, how to assign learning_ids |
| `.claude/rules/skill-metrics-protocol.md` | SKILL.metrics.md schema, who writes when, rollover policy |
| `.claude/rules/proposal-protocol.md` | Proposal doc schema, lifecycle (open → accepted/rejected), regression guard |
| `docs/meta/README.md` | Overview of the meta-learning workflow |
| `docs/meta/proposals/.gitkeep` | Initialize empty |
| `docs/meta/rejected/.gitkeep` | Initialize empty |
| `MIGRATION-PHASE-8.md` | What changed, how to roll back, no breaking changes (Phase 8 is additive) |

### Modified files
| Path | Change |
|---|---|
| `.claude/agents/evaluator.md` | Add Step: after writing each verdict, append a row to the relevant `SKILL.metrics.md`. Add Step: in retrospective mode, summarise active learning_id verification results |
| `.claude/skills/eval/output-grading/SKILL.md` | Hook for metrics-recorder invocation |
| `.claude/skills/eval/retrospective/SKILL.md` | Add section to retrospective output: "Active learning_id verifications" |
| `.claude/skills/SKILLS-CATALOG.md` | Register new skill (if Decision 8 = B) and add `meta/` section row |
| `.claude/rules/context-budget.md` | Add meta-learner row (12 files / invocation) |
| `.claude/rules/output-structure.md` | Add `docs/meta/` to the tree |
| `CLAUDE.md` | Add `/meta-learn` to command table; add fifth agent row; document meta-learning workflow |
| All existing SKILL.md files (one-time seeding) | Add empty `## Lessons Learned` section near the end |

---

## Open Decisions (need user lock-in before execution)

### Decision 1 — Trigger model

| Option | Description |
|---|---|
| A | Manual only — `/meta-learn` on demand |
| B | Manual + auto-prompt after N completed projects |
| C | Threshold-based — auto-run when any skill's first-pass FAIL rate exceeds 50% on ≥ 3 invocations |

**Recommendation:** A initially. Add B once we know what cadence is useful.

### Decision 2 — Where proposals land

| Option | Description |
|---|---|
| A | Markdown files in `docs/meta/proposals/` (consistent with current pattern) |
| B | GitHub issues |
| C | Draft PRs with the diff already applied |

**Recommendation:** A. Keeps the harness self-contained and reviewable offline. You can promote any proposal to a PR manually.

### Decision 3 — Meta-learner context scope

| Option | Allowed reads |
|---|---|
| A | Only metrics files + retrospectives + lessons + rejected |
| B | Above + project journals (routing section only) |
| C | Above + actual deliverable files (sampling) |

**Recommendation:** B. Project journals give the "why" behind retrospectives without the cost of raw deliverables.

### Decision 4 — Evidence threshold

| Option | Description |
|---|---|
| A | 1+ data point (eager) |
| B | 3+ data points across distinct projects (default) |
| C | 5+ data points (conservative) |

**Recommendation:** B. Filters noise but doesn't require waiting forever.

### Decision 5 — Rejection memory

| Option | Description |
|---|---|
| A | Fresh every run — meta-learner does not read past rejections |
| B | Reads `docs/meta/rejected/` and skips duplicates |

**Recommendation:** B. Prevents proposal spam.

### Decision 6 — Meta-learner model tier

| Option | Description |
|---|---|
| A | Opus (consistent with planner/evaluator, careful reasoning) |
| B | Sonnet (cheaper, may miss subtle patterns) |

**Recommendation:** A. Runs infrequently; pattern detection is the highest-stakes reasoning in the loop.

### Decision 7 — Metrics writing mechanism

| Option | Description |
|---|---|
| A | Evaluator agent appends to `SKILL.metrics.md` inline as part of its grading flow (no new skill) |
| B | New skill `eval/metrics-recorder` loaded by evaluator after each grading |

**Recommendation:** A. One less skill to maintain; the metrics write is a single append, not complex enough to warrant its own SKILL.md.

### Decision 8 — Seeding the `## Lessons Learned` section in existing skills

| Option | Description |
|---|---|
| A | Add the section to all 28 skills now (one-time bulk update, mostly empty placeholders) |
| B | Add lazily — only when a lesson is first recorded for a skill |

**Recommendation:** A. Consistency makes the meta-learner's parsing simpler; placeholder is one line.

---

## Phased Rollout Within Phase 8

To avoid over-engineering before we know what works, split Phase 8 into 4 sub-phases. Each is independently shippable.

| Sub-phase | Scope | Why first |
|---|---|---|
| **8.0** | Foundation files: rules (`lessons-learned-protocol`, `skill-metrics-protocol`, `proposal-protocol`), folder structure (`docs/meta/`, `.claude/lessons/`), seed `L-001` lesson, MIGRATION-PHASE-8.md | Establishes protocols before any agent depends on them |
| **8.1** | Evaluator change: append to `SKILL.metrics.md` after each grading. Add empty `## Lessons Learned` section to all 28 skills. | Start collecting telemetry passively. No new agent yet — we want real data before automating analysis. |
| **8.2** | `meta-learner` agent + `/meta-learn` command + first dry-run on whatever data has accumulated | Ship the analyst, but only after 8.1 has produced enough data points |
| **8.3** | Regression guard: retrospective skill writes "active learning_id verifications" section; meta-learner reads it next run | Closes the loop — confirms whether accepted proposals actually helped |

**Gating between sub-phases:**
- 8.0 → 8.1: rules approved
- 8.1 → 8.2: at least 3 projects' worth of metrics exist (otherwise meta-learner has nothing to analyze)
- 8.2 → 8.3: at least one proposal has been accepted and applied (so there's a learning_id to verify)

This sequencing forces us to validate each layer before building the next.

---

## Integration With the Existing Harness

### What changes in the harness loop

The harness loop in `.claude/rules/harness-loop.md` gets one additional micro-step:

```
Existing:
  c. evaluator  → loads eval/output-grading skill
                 → reads deliverables + sprint contract
                 → writes eval-verdict-<cp>-iter<N>.md (PASS or FAIL)

After Phase 8.1:
  c. evaluator  → loads eval/output-grading skill
                 → reads deliverables + sprint contract
                 → writes eval-verdict-<cp>-iter<N>.md
                 → NEW: appends row to relevant SKILL.metrics.md  (one-line write, atomic)
```

### What changes in retrospectives

After Phase 8.3, every retrospective includes:

```markdown
## Active Learning Verification
| learning_id | Skill | Expected outcome | Observed | Verdict |
|---|---|---|---|---|
| L-007 | bi/kpi-definition | C2 first-pass FAIL <20% | C2 passed first-pass | confirmed |
```

The meta-learner reads this section on its next run to decide whether to keep, modify, or revert each tracked change.

### What doesn't change

- `/project`, `/admin_resource`, `/run`, `/spec` — unchanged
- All 4 existing agent prompts — only evaluator gets a small append step
- All 28 existing skill prompts — only gain a `## Lessons Learned` placeholder
- All 3 playbooks — unchanged
- Output structure for projects — unchanged
- Sprint contracts, handoff blocks, versioning protocol — unchanged

Phase 8 is **purely additive**. There is no breaking change.

---

## Risks & Mitigations

| Risk | Mitigation |
|---|---|
| Meta-learner over-generalizes — proposes universal rules from one project's pain | Decision 4 (evidence threshold ≥ 3) + Decision 5 (scoped tiers in proposal schema) |
| Proposal pile-up — user doesn't review, proposals accumulate uselessly | Print proposal count at end of each `/meta-learn` run; CLAUDE.md documents the review cadence |
| Stale lessons — model improves; old lessons no longer apply | Each lesson has a `status` field; meta-learner can propose marking a lesson `superseded` |
| Goodhart's Law — generator learns to game first-pass PASS rate | Hard threshold PASS/FAIL kept; metrics are descriptive, not the agent's objective function |
| Cost — meta-learner reads many files | Hard read budget (12) + summary-first reading (metrics already aggregated) |
| Cross-project leakage — lesson from project A breaks project B | Regression guard (8.3) catches this within 3 invocations |
| The meta-learner itself develops blind spots | Periodic human spot-check; proposals capped at 5 per run to force prioritization |
| User rejects a good proposal, blocks future re-proposal forever | Rejection has a `permanent: true/false` field; non-permanent rejections can be re-considered with new evidence |

---

## Acceptance Criteria

Phase 8 is complete when:

- [ ] All rules files (8.0) exist and pass internal consistency check
- [ ] Every skill has a `## Lessons Learned` section (even if empty placeholder)
- [ ] Evaluator successfully appends to `SKILL.metrics.md` after each grading in a test project
- [ ] After 3 test-project runs, `SKILL.metrics.md` contains 3+ rows for at least one skill
- [ ] `/meta-learn` runs without error against the accumulated metrics
- [ ] Meta-learner produces at least one well-formed proposal document
- [ ] Meta-learner does NOT write to any file under `.claude/` (verified by audit log)
- [ ] Rejecting a proposal and moving to `docs/meta/rejected/` prevents re-proposal in subsequent runs
- [ ] After applying one accepted proposal, the next retrospective surfaces the `learning_id` verification line
- [ ] `MIGRATION-PHASE-8.md` documents rollback path (revert is a single `git revert <commit>` — Phase 8 is additive)
- [ ] `CLAUDE.md` updated with `/meta-learn`, 5-agent table, meta-learning workflow section

---

## Dependencies

- **Hard:** PR #2 (Phase 7 atomic switch) must be merged into `main` before Phase 8 starts. Phase 8 modifies `evaluator.md` and adds a fifth agent; doing this on top of v0's 29-agent architecture would create a mess.
- **Soft:** At least one real (non-dry-run) project should complete on the v1 harness before Phase 8.2 ships, so the meta-learner has actual data to analyze. Dry-run data is fine for the first end-to-end test but isn't representative.

---

## Estimated Effort

| Sub-phase | Files touched | Time estimate |
|---|---|---|
| 8.0 — Foundation | 4 new rules + 4 new structural files | 1 session |
| 8.1 — Evaluator metrics + skill seeding | 1 agent edit + 28 skill edits (mostly trivial) + 2 skill edits (eval/output-grading, eval/retrospective) | 1 session |
| 8.2 — Meta-learner agent + command | 1 new agent + 1 new command + CLAUDE.md update | 1 session |
| 8.3 — Regression guard | 1 skill edit (eval/retrospective) + meta-learner read protocol update | 0.5 session |

Total: ~3.5 sessions, depending on whether we do all four sub-phases continuously or stage them across real project runs.

---

## Open Questions for User

Before I start executing, please lock in answers for **Decisions 1–8** above. Recommendations are flagged; if you accept them all, the plan is fully specified.

Two additional things to confirm:

1. **Naming**: I'm using `meta-learner` for the agent. Alternatives: `analyst`, `reviewer-meta`, `improver`, `harness-improver`. Preference?
2. **Slash command name**: `/meta-learn` is functional but ugly. Alternatives: `/improve`, `/analyze`, `/learn`, `/review-harness`. Preference?

Once decisions are locked and naming is settled, I'll create a PR for this plan file (parallel to how `RESTRUCTURE-PLAN.md` was reviewed in PR #1) and then start executing Sub-phase 8.0 after the plan PR is approved.
