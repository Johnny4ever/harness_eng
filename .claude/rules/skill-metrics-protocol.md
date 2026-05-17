# Skill Metrics Protocol

Every skill carries an append-only telemetry sidecar that records how it performed each time the harness invoked it. The metrics file is the primary input for the `meta-learner` agent.

## File Path

```
.claude/skills/<category>/<skill>/SKILL.metrics.md
```

One sidecar per skill. Lives next to `SKILL.md` in the same directory.

## Who Writes

**Only the `evaluator` agent writes to this file.** No other agent (generator, planner, collector, meta-learner) is permitted to modify `SKILL.metrics.md`.

The evaluator appends one row after writing each `eval-verdict-<cp>-iter<N>.md` file:
- After a PASS verdict → record the row with iteration count and any failed criteria from prior iterations
- After a FAIL that hits `max_iterations` (escalation) → record the row as a non-PASS outcome

Rework iterations (FAIL → feedback → retry) do not produce a row each — only the final outcome of the checkpoint is recorded.

## Schema

```markdown
---
skill: <category>/<skill-name>
first_recorded: <YYYY-MM-DD>
total_invocations: <N>
first_pass_pass_count: <N>
first_pass_pass_rate: <0.00 – 1.00>
avg_iterations_to_pass: <decimal>
escalation_count: <N>
last_updated: <YYYY-MM-DD>
---

# Metrics: <skill-name>

## Per-Invocation Log
| Date | Project | Checkpoint | Verdict | Iter to PASS | Failed criteria | Notes |
|---|---|---|---|---|---|---|
| 2026-05-17 | sales-dash | cp1 | PASS | 2 | C2 → C0 | TBD-as-content fixed in iter 2 |

## Active Learning Tags
- L-NNN (applied <date>): "<short description>" — verify in next <N> invocations
```

### Front-matter fields
| Field | Type | Rule |
|---|---|---|
| `skill` | string | Must match the skill's canonical `name` from SKILL.md front-matter |
| `first_recorded` | date | Set on first row; never changes |
| `total_invocations` | int | Count of rows in log; recompute on every append |
| `first_pass_pass_count` | int | Rows where Iter to PASS = 1 |
| `first_pass_pass_rate` | decimal | first_pass_pass_count / total_invocations, 2 decimal places |
| `avg_iterations_to_pass` | decimal | Mean of "Iter to PASS" across PASS rows only, 1 decimal place |
| `escalation_count` | int | Rows with Verdict = ESCALATED |
| `last_updated` | date | Today on every append |

### Log row columns
| Column | Description |
|---|---|
| Date | Date of the final verdict (YYYY-MM-DD) |
| Project | Project slug |
| Checkpoint | Checkpoint id (e.g. `cp1`, `cp2`) |
| Verdict | `PASS` or `ESCALATED` |
| Iter to PASS | Number of iterations until PASS (1, 2, 3, ...). For ESCALATED, write `—` |
| Failed criteria | Arrow-separated list of failing criterion IDs per iteration (e.g. `C2,C3 → C2 → C0` means C2+C3 failed iter 1, C2 alone failed iter 2, passed iter 3). For first-pass PASS, write `—` |
| Notes | 1-line summary; reference active learning_id if relevant |

### Active Learning Tags section
When an accepted proposal applies a change to this skill, the evaluator (in retrospective mode) appends one bullet listing the `learning_id`, the change description, and how many future invocations should verify the effect (default: next 3).

The bullet is removed once verification is complete (meta-learner reads the result and either confirms or proposes revert).

## Rollover

When the log exceeds **200 rows**, the evaluator:
1. Moves the existing file to `SKILL.metrics-archive-<YYYY>.md` (year of the oldest archived row)
2. Starts a fresh `SKILL.metrics.md` carrying forward only the front-matter aggregate stats and the most recent 50 rows
3. The meta-learner reads both files when present

## What does NOT belong in this file

- Full deliverable contents
- Verbatim eval feedback text (that lives in `eval-feedback-*.md`)
- Stakeholder names or PII
- Anything the user has not seen in a verdict or retrospective

## Initialisation

`SKILL.metrics.md` is created on the **first** evaluator write for that skill. Skills do not ship with empty metrics files in the repo — they appear only after real use.

The single exception: the Phase 8.1 commit adds a placeholder header (front-matter only, empty log) for all 28 existing skills so the meta-learner can enumerate the skill set without globbing. This is the only allowed pre-populated metrics file.

## Read budget

The meta-learner reads metrics files first (cheapest, pre-aggregated). Front-matter alone gives it enough signal to decide which skills warrant deeper investigation. One full metrics file = 1 file in the read budget. Front-matter-only read = 0.25 files (per `context-budget.md`).
