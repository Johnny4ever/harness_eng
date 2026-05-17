# Context Budget

Each agent has a strict limit on how many files it may read per invocation. These limits exist to stay within token windows, prevent "context anxiety" (accumulated history degrading judgment), and enforce the discipline of structured handoffs over implicit context.

## Per-Agent File Read Limits

| Agent | Max files per invocation | What it reads |
|---|---|---|
| `collector` | 3 | `.source-registry.md` + 1 source content + `CATALOG.md` |
| `planner` | 5 | `project-journal.md` (routing section only) + `synthesis.md` + up to 3 `must_reads` from prior handoff |
| `generator` | 6 | `strategy.md` (step section only) + `sprint-contract-<cp>.md` + `SKILLS-CATALOG.md` + active skill `SKILL.md` + up to 2 `must_reads` from handoff |
| `evaluator` | 4 | `sprint-contract-<cp>.md` + primary deliverable + eval skill `SKILL.md` + prior verdict (if rework) |

Reading beyond these limits is a harness failure. If an agent believes it needs more files, it should:
1. Check if the handoff block's `must_reads` can be trimmed (indicates coupling upstream)
2. Request a smaller, more focused deliverable from the generating agent
3. Split the checkpoint into two smaller checkpoints

## What Counts as a "File Read"

- Reading a full file: **1 unit**
- Reading only the front-matter of a file: **0.25 units** (round up if content also read)
- Reading `SKILLS-CATALOG.md`: **1 unit** (always required for generator — pre-counted in budget)
- Reading a handoff block embedded in a file already read: **0** (it's part of the file)
- Re-reading a file already in context: **0** (no additional cost)

## Context Reset Triggers

A context reset (clearing accumulated history and starting fresh) happens at:

| Trigger | What is carried forward |
|---|---|
| Checkpoint boundary (any checkpoint → next checkpoint) | Handoff block from last deliverable + eval verdict |
| Agent handoff (generator → evaluator) | Handoff block only |
| Rework iteration (evaluator → generator) | Sprint contract + eval feedback file |
| Planner → generator handoff | Strategy.md step section + sprint contract for first checkpoint |
| Session resume (user returns after break) | project-journal.md (routing section) — read first |

**Do not carry forward:** raw conversation history, intermediate reasoning, prior deliverable content that is not in `must_reads`.

## Planner Context Budget Detail

The planner runs in two modes:

**Synthesis mode** (when cache is invalid or missing):
- Reads `artifacts/CATALOG.md` (1 file)
- Reads up to 4 artifact files flagged as `must_read: true` in CATALOG (4 files)
- Total: 5 files maximum

**Strategy mode** (when synthesis exists and is valid):
- Reads `synthesis.md` (1 file)
- Reads `project-journal.md` routing section only (0.5 file)
- Reads playbook front-matter to confirm selection (0.5 file)
- Total: ~2 file units

**Routing mode** (checking where the project stands):
- Reads `project-journal.md` routing section only (0.5 file)
- Reads `.cache-manifest.md` (0.5 file)
- Total: 1 file unit

## Generator Context Budget Detail

For each checkpoint step, the generator:

1. Reads `SKILLS-CATALOG.md` (1 file) — to resolve which skill to load
2. Reads the skill's `SKILL.md` (1 file) — for the step procedure
3. Reads `sprint-contract-<cp>.md` (1 file) — for acceptance criteria
4. Reads up to 2 `must_reads` from the prior step's handoff block (2 files)
5. Reads `STATUS.md` to update step status (1 file, write-back)

Total per step: **6 file reads**

The generator must NOT speculatively read files not listed in `must_reads`. If additional context is needed, it goes into the `gaps` field of the handoff block and is surfaced to the user.

## Skill SKILL.md Size Limit

Each skill file must stay under **300 lines**. If a skill exceeds this:
- Split into a parent skill + sub-skills
- Move examples to a separate `examples/` folder
- Move templates to a separate `templates/` folder

This keeps the generator's skill-read within context budget even for complex skills.

## SKILLS-CATALOG.md Size Limit

The catalog must stay under **150 lines**. It is a lookup index, not documentation. If it exceeds this:
- Abbreviate descriptions (one clause, not a sentence)
- Move resolution algorithm to its own section at the end
- Consider splitting into domain catalogs (`SKILLS-CATALOG-BI.md`, `SKILLS-CATALOG-DBT.md`) with a top-level router
