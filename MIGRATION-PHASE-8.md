# Migration Notes: Phase 8 — Meta-Learner

This is a **purely additive** phase. Nothing in the existing v1 harness behaviour changes for users who never invoke `/meta-learn`. Rollback is a single `git revert <commit>`.

## What Changed

### New files (additions only)

| Path | Purpose |
|---|---|
| `.claude/agents/meta-learner.md` | Fifth agent — analyst, never editor |
| `.claude/commands/meta-learn.md` | New slash command |
| `.claude/lessons/` | Cross-skill anti-pattern library |
| `.claude/lessons/L-001-tbd-as-sole-content.md` | Seed lesson |
| `.claude/rules/skill-metrics-protocol.md` | How `SKILL.metrics.md` files work |
| `.claude/rules/lessons-learned-protocol.md` | How embedded and library lessons interact |
| `.claude/rules/proposal-protocol.md` | Proposal doc schema and lifecycle |
| `docs/meta/proposals/` | Where meta-learner writes proposals |
| `docs/meta/proposals/accepted/` | Applied proposals (post-merge archive) |
| `docs/meta/rejected/` | Rejected proposals with reasoning |
| `docs/meta/README.md` | Workspace overview |

### Modified files

| Path | Change |
|---|---|
| `.claude/agents/evaluator.md` | Now appends a row to `SKILL.metrics.md` after each grading; in retrospective mode, summarises active learning_id verifications |
| `.claude/skills/eval/output-grading/SKILL.md` | Added Step: append metrics row after writing verdict |
| `.claude/skills/eval/retrospective/SKILL.md` | Added section: "Active Learning Verification" |
| `.claude/skills/SKILLS-CATALOG.md` | Added meta-learner row in agent table (informational) |
| `.claude/rules/context-budget.md` | Added meta-learner row (12 files / invocation) |
| `.claude/rules/output-structure.md` | Added `docs/meta/` to the tree |
| All 28 SKILL.md files | Added empty `## Lessons Learned` placeholder section |
| `CLAUDE.md` | Added `/meta-learn`, 5th agent row, meta-learning workflow section |

## What Did NOT Change

- `/project`, `/admin_resource`, `/run`, `/spec` — same behaviour
- planner, generator, collector — unchanged
- All 3 playbooks — unchanged
- All step skills (bi/, data/, dbt/, generic/, collect/) — only gained empty placeholder section
- Sprint contracts, handoff blocks, versioning protocol, context resets — unchanged
- Output structure for projects under `docs/projects/<slug>/` — unchanged

## Behavioural Changes (only when you opt in)

| When you do this | What happens |
|---|---|
| Run a project to checkpoint completion | Evaluator appends one row to the relevant `SKILL.metrics.md` files (silent, fast) |
| Run `/meta-learn` (manual invocation only) | Meta-learner reads metrics + retrospectives + journals; writes 0–5 proposals to `docs/meta/proposals/` |
| Accept a proposal | Open a PR with the diff; on merge, the change applies normally and adds a `learning_id` tag for regression tracking |
| Reject a proposal | Move file to `docs/meta/rejected/`; optionally set `permanent_reject: true` |

If you never run `/meta-learn`, the only externally visible change is the gradual growth of `SKILL.metrics.md` files. They are small (≤ 200 rows each) and ignored by every other agent.

## Rollback

To roll back Phase 8 entirely:
```bash
git revert <phase-8-merge-commit>
```

Or selectively:
```bash
# Remove the meta-learner agent and command (keeps metrics collection):
rm .claude/agents/meta-learner.md .claude/commands/meta-learn.md

# Remove metrics collection entirely (reverts evaluator change):
git checkout <pre-phase-8-tag> -- .claude/agents/evaluator.md .claude/skills/eval/output-grading/SKILL.md
```

The `SKILL.metrics.md` files themselves are harmless — they can stay as historical record even if the meta-learner is removed.

## Tagging

The pre-Phase-8 state is tagged `v1-generic-harness`. The post-Phase-8 state will be tagged `v1.1-meta-learner` once this PR merges to main.

## What This Phase Does NOT Add

Per `PHASE-8-PLAN.md` non-goals:
- No automatic application of proposals
- No continuous self-improvement loop
- No numeric skill scoring or leaderboard
- No modification of `.claude/` by any agent
- No replacement of PASS/FAIL with graded scores

The meta-learner is an analyst. You remain the editor.
