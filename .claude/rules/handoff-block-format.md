# Handoff Block Format

A handoff block is an HTML comment embedded at the end of every deliverable file. It carries structured state across context resets so the next agent can start working immediately without re-reading the full conversation.

## Schema

```
<!-- HANDOFF
from: <agent-name>
to: <next-agent-name | "evaluator" | "user-gate">
checkpoint: <checkpoint-id>
iteration: <N>
status: complete | blocked
must_reads:
  - path: <path relative to project root>
    reason: <one line — why this file is essential context>
key_findings:
  - <concise bullet — a fact the next agent must know>
gaps:
  - <unresolved issue, missing data, or open question>
do_not_redo:
  - <completed work the next agent must not repeat>
next_step: <what the receiving agent should do first>
-->
```

## Field Rules

| Field | Required | Notes |
|---|---|---|
| `from` | Yes | Name of the agent writing this block |
| `to` | Yes | Receiving agent or `"user-gate"` if awaiting human approval |
| `checkpoint` | Yes | e.g. `cp1`, `cp2`, `cp3` — matches playbook checkpoint id |
| `iteration` | Yes | Which rework iteration this is (1 = first attempt) |
| `status` | Yes | `complete` = deliverable ready for evaluation; `blocked` = cannot proceed without external input |
| `must_reads` | Yes | 1–5 files; more than 5 indicates the agent over-coupled; revisit |
| `key_findings` | Yes | 3–7 bullets; facts that would not be obvious from reading the deliverable alone |
| `gaps` | No | Leave empty only if there are genuinely zero gaps |
| `do_not_redo` | No | Use when there is a risk the next agent might re-run expensive or destructive steps |
| `next_step` | Yes | One sentence — the first concrete action the receiver should take |

## Placement

- The handoff block goes at the **very end** of the deliverable file, after all content
- One block per file; never more than one
- The block must be valid HTML comment syntax — no stray `-->` inside field values

## Example

```
<!-- HANDOFF
from: generator
to: evaluator
checkpoint: cp2
iteration: 1
status: complete
must_reads:
  - path: docs/projects/sales-dash/output/04-discovery/04-source-map.md
    reason: primary deliverable being evaluated
  - path: docs/projects/sales-dash/output/sprint-contract-cp2.md
    reason: acceptance criteria to grade against
key_findings:
  - All 5 KPIs mapped to source tables in Snowflake PROD schema
  - orders.amount field has 3% null rate — flagged in source map, within acceptable threshold
  - HK market data only available from 2023-01-01 — history constraint documented
gaps:
  - Real-time refresh feasibility unconfirmed for the live-orders KPI — needs infra team input
do_not_redo:
  - Snowflake schema scan (took 4 min, results cached in source-map appendix)
next_step: Grade the source map against all criteria in sprint-contract-cp2.md; pay attention to the HK history constraint — evaluating whether it meets the 2-year history requirement.
-->
```

## Reading a Handoff Block

When an agent receives a handoff:
1. Read the block first — before reading any other file
2. Load only the `must_reads` files — do not explore beyond them unless the block has gaps requiring investigation
3. Act on `do_not_redo` — treat those items as settled
4. Address `gaps` proactively — if the gap blocks you, escalate immediately rather than producing a partial deliverable
5. Begin with the `next_step` action

## Writing a Handoff Block

Before writing the block, the agent must:
1. Confirm all deliverable files are written and saved
2. Self-check against the sprint contract checklist (if applicable)
3. Identify the exact `must_reads` — only files the receiver genuinely needs, not everything you read
4. Be honest about `gaps` — omitting a known gap that later blocks the receiver is a harness failure
