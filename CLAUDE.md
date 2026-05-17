# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

A **reusable Claude Code agent harness template** built on the Anthropic planner/generator/evaluator pattern. There is no application code — the entire repo is a `.claude/` folder containing agent definitions, slash commands, playbooks, and skills that can be dropped into any project to enable structured analytical and data engineering workflows.

| Command | Entry agent | What it does |
|---|---|---|
| `/project` | `planner` | Full project lifecycle: classify intent → plan → generate → evaluate → retrospective |
| `/admin_resource` | `collector` | Ingest, catalog, search, and version project knowledge artifacts |
| `/run <playbook>` | `generator` | Direct playbook execution — power-user escape hatch, skips planner |
| `/spec` | *(inline)* | Scaffold a feature spec file and git branch from a short idea |

To deploy this harness into a project, copy the `.claude/` folder into the project root.

---

## Agent Architecture

4 agents in `.claude/agents/`. Each is a Claude Code native agent with YAML front-matter (`name`, `description`, `model`) followed by a system prompt.

| Agent | Model | Role |
|---|---|---|
| `planner` | `claude-opus-4-7` | Classify intent, synthesize artifact library, select playbook, produce execution plan |
| `generator` | `claude-sonnet-4-6` | Execute playbook steps, load skills, produce deliverables, self-check |
| `evaluator` | `claude-opus-4-7` | Write sprint contracts, grade PASS/FAIL, synthesize feedback, run retrospective |
| `collector` | `claude-sonnet-4-6` | Ingest, catalog, search, and version project knowledge artifacts |

### Harness Loop

```
/project → planner classifies intent
         → planner selects playbook + produces strategy.md
         → human gate: user approves plan

For each checkpoint in the playbook:
  evaluator writes sprint contract (observable PASS/FAIL criteria)
  generator executes steps → produces deliverables
  evaluator grades: PASS or FAIL
    FAIL → evaluator writes feedback brief → generator reworks → repeat (max 5 iterations)
    PASS → human gate: user approves before next checkpoint

After final checkpoint:
  evaluator writes retrospective → stored to artifacts/ad-hoc/
```

Context resets between checkpoints via structured HANDOFF blocks (`<!-- HANDOFF: ... -->`).

---

## Skills

28 composable skills in `.claude/skills/`, organized by domain. Skills are NOT invoked by users — they are loaded by the generator (and collector) when a playbook step requires them.

The generator finds skills via **`.claude/skills/SKILLS-CATALOG.md`** — a single lookup table. It never globs SKILL.md files directly.

Resolution order: native Claude Code plugin skill → local `.claude/skills/<path>/SKILL.md` → escalate to user.

### Skill categories

| Category | Skills |
|---|---|
| `bi/` | kpi-definition, wireframe-ux, dashboard-build, validation |
| `data/` | discovery, quality-profiling, semantic-modeling |
| `dbt/` | model-build, test-design, documentation |
| `generic/` | requirement-intake, stakeholder-alignment, documentation, governance-check, source-enablement, release-checklist |
| `eval/` | contract-definition, output-grading, feedback-synthesis, retrospective |
| `collect/` | ingest-confluence, ingest-jira, ingest-adhoc, catalog-index, version-supersede, search |

---

## Playbooks

3 playbooks in `.claude/playbooks/`. A playbook is a recipe — it defines step sequence, parallelisation, checkpoint gates, cross-cutting skills, and output folder structure for a domain.

| Playbook | Steps | Checkpoints | Use when |
|---|---|---|---|
| `bi-dashboard` | 12 | 4 | Delivering an end-to-end BI dashboard (req → wireframe → build → QA → release) |
| `dbt-data-product` | 10 | 3 | Delivering a curated dbt mart layer (no BI layer) |
| `kpi-proof` | 1 | 1 | Lightweight: prove the harness loop works with a single KPI definition step |

---

## State Persistence

All state is stored as Markdown files — no database. Key files to read when picking up a project mid-flight:

```
docs/projects/<slug>/
  project-journal.md        ← canonical project state (planner writes, reads on every invocation)
  DECISIONS.md              ← version evolution log
  output/                   ← deliverables in step-numbered subfolders (01-requirement/, 02-kpi/, ...)
  output/STATUS.md          ← current step, checkpoint, iteration, blockers
  output/sprint-contract-cp<N>.md
  output/eval-verdict-cp<N>-iter<M>.md
  output/eval-feedback-cp<N>-iter<M>.md

artifacts/
  CATALOG.md                ← master knowledge index (collector maintains)
  CHANGELOG.md              ← artifact evolution log
  .source-registry.md       ← MCP server → source_type mapping
  confluence/ jira/ ad-hoc/ ← topic-based artifact files (ART-YYYYMMDD-NNN-<slug>.md)
```

---

## Key Operational Concepts

**Human gates** — Every checkpoint requires explicit user approval before the generator advances. The planner presents a summary of what was produced and stops.

**Sprint contracts** — The evaluator writes observable, binary PASS/FAIL criteria *before* the generator produces any output for a checkpoint. This makes grading unambiguous.

**Hard-threshold grading** — The evaluator defaults to FAIL. Partial completion = FAIL. Placeholders (TBD as only field content) = FAIL. Effort is not weighted.

**Handoff blocks** — Files include `<!-- HANDOFF: ... -->` blocks with key findings, gaps, and `must_reads` for the next agent. Always read and honor these.

**Context budgets** — Per-agent read limits: planner=5 files, generator=6, evaluator=4, collector=3. Skills are capped at 300 lines each.

**Versioning protocol** — Before overwriting any deliverable, archive to `versions/<filename>-v<N>-<YYYYMMDD>.md` and append to `DECISIONS.md`.

**Iteration budget** — Max 5 rework cycles per checkpoint. On budget exhaustion, the generator escalates to the user rather than continuing silently.

---

## Modifying the Harness

- **Add a skill**: create `.claude/skills/<category>/<name>/SKILL.md` following the YAML front-matter schema, then add a row to `SKILLS-CATALOG.md`.
- **Add a playbook**: create `.claude/playbooks/<name>.md` with steps, checkpoints, and parallelisation rules following the existing playbook pattern.
- **Change agent behavior**: edit `.claude/agents/<name>.md` system prompt.
- **Change model tier**: edit the `model:` front-matter field.
- **Change output paths or schemas**: edit the relevant `.claude/rules/<name>.md` file.

---

## MCP Integration

The collector agent discovers available MCP servers on each ingestion run. Supported source types: `confluence`, `jira`, `ad-hoc`. Non-updateable types (`ad-hoc`) are skipped by `/admin_resource update`. The MCP → source_type mapping is persisted in `artifacts/.source-registry.md`.
