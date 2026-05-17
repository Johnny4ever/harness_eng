# Meta-Learning Workspace

This folder is where the `meta-learner` agent writes proposals and where you keep the rejection log. Nothing here is read or modified by the main harness loop (`/project`, `/run`, `/admin_resource`).

## Folders

| Folder | Contents |
|---|---|
| `proposals/` | Open proposals awaiting your review |
| `proposals/accepted/` | Proposals that have been applied via PR (kept for history) |
| `rejected/` | Proposals you rejected, with `## Rejection Reasoning` appended |

## How it works

1. After several real projects have completed, run `/meta-learn`
2. The agent reads accumulated metrics (`.claude/skills/**/SKILL.metrics.md`), retrospectives, and project journals
3. It identifies recurring patterns (≥ 3 distinct projects per pattern, max 5 proposals per run)
4. It writes proposals to `proposals/` — never to `.claude/`
5. You review each proposal:
   - **Accept** → open a PR applying the diff in the Proposed Change section. On merge, move the file to `proposals/accepted/`.
   - **Reject** → move the file to `rejected/` and append a `## Rejection Reasoning` section. Set `permanent_reject: true` in front-matter to block re-proposal.
   - **Defer** → leave it; next `/meta-learn` run skips it.

## Schemas

- Proposal schema: `.claude/rules/proposal-protocol.md`
- Metrics schema: `.claude/rules/skill-metrics-protocol.md`
- Lessons protocol: `.claude/rules/lessons-learned-protocol.md`

## What this is NOT

- Not a queue the harness consumes automatically — there is no auto-apply
- Not a dashboard — the harness is markdown, not a UI
- Not always-on — the meta-learner only runs when you invoke `/meta-learn`
