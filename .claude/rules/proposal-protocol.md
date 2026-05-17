# Proposal Protocol

The `meta-learner` agent writes **proposal documents** when it finds a recurring pattern that suggests a skill, agent, rule, or playbook should change. Proposals are never auto-applied — the human reviews each one and either accepts (creates a normal git PR with the diff) or rejects (moves the file to `docs/meta/rejected/`).

This protocol defines the proposal schema, lifecycle, and review workflow.

## File Paths

| State | Path |
|---|---|
| Open (awaiting review) | `docs/meta/proposals/<YYYY-MM-DD>-<NNN>-<short-slug>.md` |
| Rejected | `docs/meta/rejected/<YYYY-MM-DD>-<NNN>-<short-slug>.md` |
| Accepted | Moved into `docs/meta/proposals/accepted/<YYYY-MM-DD>-<NNN>-<short-slug>.md` after the implementing PR merges |

Filename: date + sequence number (per-day) + slug. Example: `2026-08-22-001-bi-kpi-grain-disambiguation.md`.

## Schema

```markdown
---
proposal_id: PROP-<YYYY-MM-DD>-<NNN>
created: <YYYY-MM-DD>
created_by: meta-learner
target: <path under .claude/ that would be modified>
target_type: skill | agent | rule | lesson | playbook
scope: project-local | domain | universal
status: open | accepted | rejected | superseded
evidence_count: <N>
related_lesson_id: L-NNN | null
permanent_reject: false   # set true only when user rejects with reasoning
---

# Proposal: <one-line title>

## Evidence
<bulleted list of concrete observations: project slug, checkpoint, what failed, with citation>

Pattern: <1–2 sentences naming the recurring failure>

## Hypothesis
<2–4 sentences on the most likely root cause>

## Proposed Change
<Concrete diff or new file content — ready to paste into a PR>

## Expected Outcome
<Measurable prediction: which metric should change, by how much, over what horizon>

## Tracking (learning_id)
<If accepted, tag with learning_id L-NNN. Next <N> invocations of <target> auto-include verification line.>

## Rollback Criterion
<Observable condition that would trigger a revert proposal>

## Risk Assessment
<Low | Medium | High, plus 1–2 sentences>

## Alternatives Considered
<Other approaches the meta-learner considered and rejected, with reasoning>
```

### Field rules

| Field | Rule |
|---|---|
| `proposal_id` | Globally unique. Format: `PROP-<YYYY-MM-DD>-<NNN>`. Sequence resets daily. |
| `target` | Single file path under `.claude/`. If the change touches multiple files, file separate proposals or use one proposal with a clear primary target |
| `target_type` | One of the five enums; meta-learner classifies based on path |
| `scope` | Forces the meta-learner to argue how widely the lesson applies; reviewer can challenge |
| `evidence_count` | Number of distinct projects in the Evidence section. Must be ≥ 3 (per Decision 4) |
| `related_lesson_id` | If the proposal creates or references a lesson, link the ID |
| `permanent_reject` | Default false; set true when the user rejects with "do not propose this again" |

### Section rules

**Evidence**: every bullet must cite project slug + checkpoint + observable evidence (a verdict file, a metrics row). No "I noticed" or "in general."

**Proposed Change**: the diff must be a complete, paste-ready edit — not "consider adding something about X." If the meta-learner cannot produce a concrete diff, it should not file a proposal.

**Expected Outcome**: must be a measurable claim ("C2 first-pass FAIL rate drops from 60% to <20% over next 3 invocations"). Vague predictions ("things should improve") are rejected.

**Rollback Criterion**: a sentence the meta-learner can later evaluate to decide whether to file a revert proposal.

**Alternatives Considered**: forces the meta-learner to argue against other reasonable interpretations of the evidence. Cannot be empty.

## Lifecycle

```
1. meta-learner runs (manual /meta-learn invocation)
2. Identifies a pattern with evidence_count ≥ 3 across distinct projects
3. Checks docs/meta/rejected/ for prior proposals on the same target with permanent_reject: true
4. If not blocked: writes a new proposal to docs/meta/proposals/
5. Reports proposal_id and path to the user

User review:
6a. ACCEPT path:
    - User opens a git PR applying the diff from the Proposed Change section
    - PR adds a `## Lessons Learned` entry to the affected skill with the proposal_id and a new learning_id
    - If the lesson is cross-cutting, PR adds a new file in .claude/lessons/
    - On merge, the proposal file is moved to docs/meta/proposals/accepted/<filename>
    - The evaluator's next invocation of the affected skill adds an "Active Learning Tags" bullet to that skill's SKILL.metrics.md

6b. REJECT path:
    - User moves the proposal file to docs/meta/rejected/
    - User appends a "## Rejection Reasoning" section explaining why
    - If the user wants to permanently block re-proposal, they edit `permanent_reject: true` in the front-matter

6c. DEFER path:
    - User leaves the file in docs/meta/proposals/ (no action)
    - Next meta-learner run skips it (already exists) but reports it in the open-proposals summary
```

## Regression Guard

When an accepted proposal lands:
1. The applied skill's `SKILL.metrics.md` gains an Active Learning Tags entry citing the `learning_id` and expected outcome
2. The next 3 invocations of that skill auto-include a verification line in their retrospective
3. The meta-learner's next run reads those verifications and:
   - If all 3 confirm the expected outcome → removes the active tag; lesson confirmed
   - If 2+ contradict the expected outcome → files a revert proposal with `target` = the same file and reasoning "L-NNN did not improve the metric as predicted"

## Proposal Cap

Each `/meta-learn` run is capped at **5 proposals**. If the meta-learner identifies more than 5 candidates, it ranks them by:
1. Evidence count (more is better)
2. Scope (universal > domain > project-local)
3. Estimated impact (drawn from Expected Outcome)

and writes only the top 5. The remaining candidates are noted in the run summary but not filed.

This forces prioritization and prevents proposal pile-up.

## Forbidden Behaviour

| Behaviour | Why forbidden |
|---|---|
| Meta-learner writing directly to `.claude/` | Defeats the audit trail; violates the analyst-not-editor rule |
| Filing a proposal with `evidence_count < 3` | Violates Decision 4; produces noise |
| Filing a proposal that duplicates an existing open proposal | Forces user to dedupe; spam |
| Filing a proposal that duplicates a `permanent_reject: true` rejected proposal | Ignores prior decision |
| Proposing a change without a paste-ready diff | Pushes synthesis work onto the human |
| Filing a proposal whose target is itself in `.claude/agents/meta-learner.md` | Loop hazard; meta-learner cannot propose changes to itself |

## Self-Check (meta-learner)

Before writing each proposal, the meta-learner confirms:

- [ ] `evidence_count` ≥ 3 distinct projects (not 3 invocations on one project)
- [ ] No existing open proposal on the same `target` with overlapping rationale
- [ ] No rejected proposal with `permanent_reject: true` on the same `target`
- [ ] The Proposed Change is a paste-ready diff, not a sketch
- [ ] Expected Outcome contains a measurable claim
- [ ] Alternatives Considered is non-empty
- [ ] Target is NOT `.claude/agents/meta-learner.md`
