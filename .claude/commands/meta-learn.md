# Meta-Learn (`/meta-learn`)

You were invoked via the **`/meta-learn`** slash command.

Route to the **`meta-learner` agent** (`.claude/agents/meta-learner.md`). The meta-learner reads accumulated harness telemetry and writes evidence-backed improvement proposals to `docs/meta/proposals/`. It never modifies `.claude/` itself.

## Usage

```
/meta-learn
```

No arguments. The agent decides what to read based on current state.

## What it does

1. Scans every `.claude/skills/**/SKILL.metrics.md` front-matter for skills meeting the trigger threshold (≥ 3 invocations AND any of: first-pass PASS rate < 50%, avg iterations > 2.0, ≥ 1 escalation)
2. Deep-reads up to 3 flagged metrics files, up to 4 retrospectives, up to 3 project journals
3. Checks `docs/meta/rejected/` for permanently rejected proposals on the same targets
4. Drafts up to 5 proposals, each with: evidence, hypothesis, paste-ready diff, expected outcome, rollback criterion, risk assessment, alternatives considered
5. Writes proposals to `docs/meta/proposals/<YYYY-MM-DD>-<NNN>-<slug>.md`
6. Prints a run summary to the conversation

## What it does NOT do

- Does NOT modify any file under `.claude/`
- Does NOT auto-apply any proposal
- Does NOT propose changes to its own agent file
- Does NOT file more than 5 proposals per run
- Does NOT file proposals with < 3 distinct projects as evidence

## Refusal Conditions

If any of these are true, the meta-learner prints a data-shortage notice and exits without writing any proposal:

- Fewer than 3 distinct projects have completed any checkpoint
- No `SKILL.metrics.md` files contain any rows
- No retrospective artifacts exist in `artifacts/ad-hoc/`

## Review Workflow

After the meta-learner reports the proposal list:

1. Open each proposal file under `docs/meta/proposals/`
2. Read the Evidence + Hypothesis + Proposed Change carefully
3. Choose one of three actions:

| Action | What to do |
|---|---|
| **Accept** | Open a git PR applying the diff in the Proposed Change section. The PR should also add a `## Lessons Learned` entry to the affected skill citing the proposal_id and a fresh learning_id. On merge, move the proposal file to `docs/meta/proposals/accepted/`. |
| **Reject** | Move the proposal file to `docs/meta/rejected/` and append a `## Rejection Reasoning` section explaining why. Set `permanent_reject: true` in the front-matter if you want to block re-proposal. |
| **Defer** | Leave the file in `docs/meta/proposals/`. The next `/meta-learn` run will skip it (already exists) but note it in the open-proposals count. |

## Regression Guard

When an accepted proposal lands:
- The applied skill's `SKILL.metrics.md` gains an `Active Learning Tags` entry citing the learning_id
- The next 3 retrospectives include an `Active Learning Verification` row evaluating whether the change met its Expected Outcome
- The next `/meta-learn` run reads these verifications and:
  - Removes confirmed tags (lesson absorbed)
  - Files a revert proposal if `contradicted` outcomes outnumber `confirmed` ones

## Output Files

After a run completes, you will find:
- New proposal files in `docs/meta/proposals/<YYYY-MM-DD>-<NNN>-<slug>.md` (one per filed proposal)
- A console summary listing each proposal_id, path, and the categories of skipped candidates
- No changes to anything under `.claude/`

## Suggested Cadence

The meta-learner is not on a schedule. Run it when:
- 3+ projects have completed since the last run
- A specific skill has been flagged repeatedly in retrospectives
- You want a checkpoint on harness health before adding new skills or playbooks

Running it too often produces noise (insufficient new evidence between runs). Running it never wastes the telemetry that's being collected.

## Integration With Other Agents

The meta-learner is the only agent that crosses project boundaries. The planner, generator, evaluator, and collector all scope to a single project per invocation. The meta-learner explicitly looks across projects to find shared patterns.

It does not call any other agent. It does not invoke any skill from `SKILLS-CATALOG.md`. Its system prompt contains everything it needs.
