# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

A **reusable Claude Code agent harness template**. There is no application code — the entire repo is a `.claude/` folder containing agent definitions, slash commands, and permissions that can be dropped into any project to enable three slash-command-driven workflows:

| Command | Entry agent | What it does |
|---|---|---|
| `/project` | `project-dispatcher` | Full project lifecycle: ingest knowledge → synthesize → plan → execute → review |
| `/bi_agent` | `bi-orchestrator` | End-to-end BI dashboard delivery (14 steps, 3 human-gated checkpoints) |
| `/admin_resource` | `resource-orchestrator` | Ingest, catalog, search, and version project knowledge artifacts |
| `/spec` | *(inline)* | Scaffold a feature spec file and git branch from a short idea |

To deploy this harness into a project, copy the `.claude/` folder into the project root. No package installation or build step required.

---

## Agent Architecture

All 25 agents live in `.claude/agents/`. Each file is a Claude Code native agent with YAML front-matter (`name`, `description`, `model`) followed by a system prompt. Model tier is assigned by role:

| Tier | Model | Used for |
|---|---|---|
| Opus | `claude-opus-4-7` | Orchestrators and planners that must reason across many artifacts |
| Sonnet | `claude-sonnet-4-6` | Builders, writers, SQL, QA, and documentation agents |
| Haiku | `claude-haiku-4-5-20251001` | High-frequency lookup agents (search, registry) |

### Group 1 — Project Intelligence (4 agents)

Single entry point: `project-dispatcher` classifies intent and routes; never call specialist agents directly.

```
project-dispatcher (Opus)
  ├─ project-synthesizer (Opus)   reads artifact library → produces synthesis.md
  ├─ project-strategist  (Opus)   reads synthesis → produces strategy.md + journal
  └─ project-reviewer    (Sonnet) reads executor output → produces lesson-learned artifacts
```

The dispatcher reads at most 2 files per invocation (context budget). It consults `docs/.cache-manifest.md` and the routing section of `docs/projects/<slug>/project-journal.md` to decide whether synthesis is needed or can be skipped.

### Group 2 — BI Agents (15 agents)

`bi-orchestrator` drives a **14-step, 3-checkpoint** pipeline with mandatory user-approval gates between checkpoints:

```
Checkpoint 1 (user reviews before proceeding)
  01  bi-requirement-intake
  02  bi-kpi-metric-definition
  03  bi-stakeholder-alignment

Checkpoint 2 (user reviews before proceeding)
  04  bi-data-discovery
  05  bi-data-quality-profiling
  06  bi-semantic-model-design
  07  bi-transformation-sql-build

Checkpoint 3 (user reviews before proceeding)
  08  bi-wireframe-ux

Post-checkpoint (runs when user is ready)
  09  bi-build
  10  bi-validation-qa
  11  bi-documentation-knowledge
  12  bi-release-deployment

Cross-cutting (invoked at any stage when needed)
  13  bi-source-enablement    — unblocks data access gaps
  14  bi-governance-reuse     — enforces enterprise KPI/naming standards
```

Steps 04 and 02 can run in parallel after step 01 completes. The orchestrator must not advance past a checkpoint until the user explicitly approves.

### Group 3 — Resource Admin (6 agents)

`resource-orchestrator` routes all `/admin_resource` sub-commands:

```
resource-orchestrator (Opus)
  ├─ resource-registry   (Haiku)  — MCP server discovery + artifact ID sequencing
  ├─ resource-ingestor   (Sonnet) — decomposes source content into topic-based artifact files
  ├─ resource-cataloger  (Sonnet) — maintains CATALOG.md, detects relationships/duplicates
  ├─ resource-search     (Haiku)  — natural-language search over the artifact library
  └─ resource-versioner  (Sonnet) — manages supersession chains + CHANGELOG
```

---

## State Persistence

All state is stored as Markdown files — no database. These are the key files to read when picking up a project mid-flight:

```
docs/
  .cache-manifest.md              ← synthesis cache validity (read by dispatcher)
  projects/<slug>/
    project-journal.md            ← canonical project state — read this first
    DECISIONS.md                  ← evolution log (every version bump appended here)
    synthesis/synthesis.md        ← latest artifact synthesis
    strategy/strategy.md          ← approved execution plan
    output/                       ← BI deliverables, step-numbered subfolders 01–14

artifacts/
  CATALOG.md                      ← master knowledge index
  CHANGELOG.md                    ← artifact evolution log
  .source-registry.md             ← MCP server → source_type mapping
  confluence/ jira/ teams-chat/ meeting-transcript/ ad-hoc/
```

---

## Key Operational Concepts

**Human gates** — Both `/project` and `/bi_agent` pause at defined checkpoints and must not auto-advance. The orchestrator presents artifacts for review and waits for explicit user approval.

**Handoff blocks** — Deliverable files include an HTML comment block (`<!-- HANDOFF: ... -->`) with key findings, gaps, and `must_reads` for the next agent. Always read and honor these before continuing from a prior artifact.

**Context budgets** — Each agent spec defines how many files it may read per invocation. Respect these limits; they exist to stay within token windows.

**Universal versioning protocol** — Before overwriting any deliverable, archive it to a `versions/` subfolder and append a one-line entry to `DECISIONS.md`. A `VERSION-INDEX.md` tracks all versions. Every step agent performs its own version bump — the orchestrator verifies it happened.

**Material vs. non-material changes** — Agents must distinguish material changes (new data sources, shifted KPIs, scope changes) that require downstream re-runs from editorial changes that do not.

**Iteration budget** — `/project execute` is capped at 3 iterations. If a project exceeds budget, escalate to the user rather than continuing silently.

---

## Modifying Agents

Each agent file follows a consistent structure:
```
role definition → role boundaries → inputs → responsibilities → output schema → success criteria
```

- To change **what an agent does**: edit its `.claude/agents/<name>.md` system prompt.
- To change **when an agent is invoked**: edit its `description:` field — Claude Code uses this for automatic invocation matching.
- To change **output file paths or schemas**: update the relevant `rules/` file referenced in the agent prompt (e.g. `rules/bi-output-structure.md`). The rules files are not in this template — they live in the consuming project.
- To change **model tier**: edit the `model:` front-matter field using the tiers above.

---

## MCP Integration

`resource-registry` dynamically discovers available MCP servers on each ingestion run and maps source URLs to server names in `artifacts/.source-registry.md`. Supported source types: `confluence`, `jira`, `teams-chat`, `meeting-transcript`, `ad-hoc`. Non-updateable types (`ad-hoc`, `teams-chat`, `meeting-transcript`) are skipped by `/admin_resource update`.
