# Harness Restructure Plan: Agents → Generic Loop + Skills + Playbooks

**Status:** Decisions locked (2026-05-16) — awaiting user "go" command before execution starts at Phase 0
**Goal:** Convert the current 25-agent, domain-specific harness into a generic 4-agent harness (Planner / Generator / Evaluator / Collector) with composable Skills and per-domain Playbooks. The result must handle BI dashboards, dbt data products, deep-dive analysis, and arbitrary future workflows under the same machinery.

---

## 1. Target Architecture

### 1.1 The 4 Agents (the machine)

Each agent is **domain-agnostic**. None of them know what a KPI or a dbt model is.

| Agent | Model | Role |
|---|---|---|
| `collector` | Sonnet | Ingest source material into the artifact library. Replaces the 6-agent resource group. Loads skills for ingestion, cataloging, search, versioning. |
| `planner` | Opus | Read artifacts, synthesize project state, select a playbook, produce a strategy doc + journal. Replaces `project-dispatcher`, `project-synthesizer`, `project-strategist`. |
| `generator` | Sonnet | Execute steps from the playbook by loading the named skill, producing deliverables, self-checking, handing off. Replaces `bi-orchestrator` + 14 BI step agents. |
| `evaluator` | Opus | Define sprint contracts, grade generator output with hard thresholds, synthesize feedback, manage the rework loop, and run retrospectives. Replaces `bi-validation-qa` (subjective parts) and `project-reviewer`. |

Each agent file is short (~100–200 lines): role, role boundaries, loop shape, handoff format, how to load skills. The domain knowledge lives in skills, not agents.

### 1.2 Skills (the knowledge)

Skills are reusable, opinionated capability files. Folder layout:

```
.claude/skills/
  SKILLS-CATALOG.md       ← fast-lookup index, hybrid resolution registry, reverse index
  bi/
    kpi-definition/
    wireframe-ux/
    dashboard-build/
    semantic-model-design/
  data/
    discovery/
    quality-profiling/
    semantic-modeling/      ← shared by BI + dbt playbooks
  dbt/
    model-build/
    test-design/
    documentation/
  eval/
    contract-definition/
    output-grading/
    feedback-synthesis/
    retrospective/
  collect/
    ingest-confluence/
    ingest-jira/
    ingest-adhoc/
    catalog-index/
    version-supersede/
    search/
  generic/
    requirement-intake/
    stakeholder-alignment/
    release-checklist/
    documentation/
    governance-check/
    source-enablement/
```

Each skill folder contains:

```
<skill-name>/
  SKILL.md              ← YAML front-matter + procedure body
  templates/            ← deliverable templates (optional)
  examples/             ← good and bad output examples (optional)
  checklist.md          ← self-eval checklist run before handoff (optional)
```

#### Skill catalog (`SKILLS-CATALOG.md`)

The catalog is the **single lookup point** the generator reads before invoking any skill. Without it, the generator would glob ~28 SKILL.md files and parse their front-matter on every dispatch — too expensive on context budget.

The catalog has three sections:

1. **Local skills table** — every skill under `.claude/skills/<category>/<name>/`, with columns: path, category, purpose, trigger keywords, playbooks that consume it.
2. **Native skills table** — Claude Code native plugin skills (e.g., `anthropic-skills:docx`, `engineering:documentation`, `atlassian:capture-tasks-from-meeting-notes`) that overlap or complement local skills.
3. **Resolution algorithm** — explicit precedence rules for the hybrid native+local lookup (Decision 1).

**Resolution order at runtime:**
1. If the playbook step names a skill explicitly (path or native name) → use it
2. Else match step's `intent_tags` against the catalog's Trigger keywords column
3. If both native and local match, default to native unless playbook overrides
4. If no match, escalate to user

**Maintenance protocol:** any commit that adds, renames, or removes a skill MUST update `SKILLS-CATALOG.md` in the same commit. Mirrors how `artifacts/CATALOG.md` is maintained for artifacts. Phase 4 builds the initial catalog; subsequent phases keep it in sync.

### 1.3 Playbooks (the recipe)

A playbook is a domain-specific assembly of skills with sequencing, gates, and output structure.

```
.claude/playbooks/
  bi-dashboard.md           ← the current 14-step BI workflow
  dbt-data-product.md       ← discover → model → test → document → release
  analysis-deep-dive.md     ← intake → discover → analyze → narrative → review
  spec-to-build.md          ← spec → plan → build → eval (engineering work)
```

Each playbook declares:

- Ordered step list, each referencing a skill by path
- Parallelization rules (which steps can run concurrently)
- Human gate positions (where to pause for user approval)
- Eval contract checkpoints (where `evaluator` defines acceptance criteria)
- Output folder structure (the playbook owns this, not the generator)
- Cross-cutting skills (e.g., `source-enablement`, `governance-check`) and when they're invoked

### 1.4 Commands

**Principle: the user describes intent; the harness picks the playbook and loads skills.** Users never name a skill or playbook in normal use. Adding a domain-specific command for every playbook (`/bi_agent`, `/dbt_agent`, ...) would defeat the entire restructure.

Final command surface after migration:

| Command | Routes to | Behavior |
|---|---|---|
| `/project <intent>` | `planner` | **Primary entry.** Planner classifies intent, picks playbook, hands off to generator + evaluator loop. |
| `/admin_resource <sub-cmd>` | `collector` | Unchanged sub-commands: ingest, catalog, search, version. |
| `/spec <idea>` | inline | Unchanged — feature spec scaffolding. |
| `/run <playbook> <context>` | `generator` (escape hatch) | Power-user override: skip planner, force a specific playbook. Used for: testing new playbooks during dev, overriding when intent classification is ambiguous, raw-context invocations. **Not the recommended path.** |

**Deprecated and removed at Phase 7:**

| Command | Replacement |
|---|---|
| ~~`/bi_agent`~~ | `/project <intent>` (planner auto-routes to `bi-dashboard` playbook). For explicit override: `/run bi-dashboard <context>`. |

Removing `/bi_agent` is a deliberate breaking change. Users with muscle memory will get a "command not found" message during Phase 7 — that's acceptable because the replacement (`/project`) is already the recommended path today and accepts the same natural-language input.

### 1.5 Rules (harness invariants)

Shared protocols promoted to top-level `rules/` (not in any agent or skill):

```
.claude/rules/
  harness-loop.md            ← the generate → eval → rework loop spec
  handoff-block-format.md    ← <!-- HANDOFF: ... --> schema
  sprint-contract-schema.md  ← what an eval contract file looks like
  versioning-protocol.md     ← archive → versions/ + DECISIONS.md append
  output-structure.md        ← docs/projects/<slug>/ layout
  context-budget.md          ← per-agent read limits, context-reset rules
```

---

## 2. Component Inventory

### 2.1 Retire (move content into skills, then delete file)

| Current file | Becomes |
|---|---|
| `.claude/agents/bi-orchestrator.md` | Logic absorbed into `generator.md` + `playbooks/bi-dashboard.md` |
| `.claude/agents/bi-requirement-intake.md` | `skills/generic/requirement-intake/SKILL.md` (generalize) |
| `.claude/agents/bi-kpi-metric-definition.md` | `skills/bi/kpi-definition/SKILL.md` |
| `.claude/agents/bi-stakeholder-alignment.md` | `skills/generic/stakeholder-alignment/SKILL.md` |
| `.claude/agents/bi-data-discovery.md` | `skills/data/discovery/SKILL.md` |
| `.claude/agents/bi-data-quality-profiling.md` | `skills/data/quality-profiling/SKILL.md` |
| `.claude/agents/bi-semantic-model-design.md` | `skills/data/semantic-modeling/SKILL.md` |
| `.claude/agents/bi-transformation-sql-build.md` | `skills/dbt/model-build/SKILL.md` |
| `.claude/agents/bi-wireframe-ux.md` | `skills/bi/wireframe-ux/SKILL.md` |
| `.claude/agents/bi-build.md` | `skills/bi/dashboard-build/SKILL.md` |
| `.claude/agents/bi-validation-qa.md` | Split: production-safety parts → `skills/generic/release-checklist/SKILL.md`; subjective grading → `skills/eval/output-grading/SKILL.md` |
| `.claude/agents/bi-release-deployment.md` | `skills/generic/release-checklist/SKILL.md` |
| `.claude/agents/bi-documentation-knowledge.md` | `skills/generic/documentation/SKILL.md` |
| `.claude/agents/bi-source-enablement.md` | `skills/generic/source-enablement/SKILL.md` |
| `.claude/agents/bi-governance-reuse.md` | `skills/generic/governance-check/SKILL.md` |
| `.claude/agents/project-dispatcher.md` | Logic merged into `planner.md` (intent classification) |
| `.claude/agents/project-synthesizer.md` | Logic merged into `planner.md` (synthesis phase) |
| `.claude/agents/project-strategist.md` | Logic merged into `planner.md` (strategy phase) |
| `.claude/agents/project-reviewer.md` | `skills/eval/retrospective/SKILL.md` (called by `evaluator`) |
| `.claude/agents/resource-orchestrator.md` | Logic merged into `collector.md` |
| `.claude/agents/resource-ingestor.md` | `skills/collect/ingest-*/SKILL.md` (one per source type) |
| `.claude/agents/resource-cataloger.md` | `skills/collect/catalog-index/SKILL.md` |
| `.claude/agents/resource-search.md` | `skills/collect/search/SKILL.md` |
| `.claude/agents/resource-registry.md` | `skills/collect/registry-resolve/SKILL.md` |
| `.claude/agents/resource-versioner.md` | `skills/collect/version-supersede/SKILL.md` |

**Result: 25 agents → 4 agents.**

### 2.2 Create

```
.claude/agents/
  collector.md             ← NEW
  planner.md               ← NEW
  generator.md             ← NEW
  evaluator.md             ← NEW

.claude/playbooks/
  bi-dashboard.md          ← NEW (reproduces current 14-step BI flow)
  dbt-data-product.md      ← NEW
  analysis-deep-dive.md    ← NEW

.claude/rules/
  harness-loop.md          ← NEW
  handoff-block-format.md  ← NEW
  sprint-contract-schema.md ← NEW
  versioning-protocol.md   ← migrated from existing implicit rules
  output-structure.md      ← migrated
  context-budget.md        ← NEW

.claude/skills/
  SKILLS-CATALOG.md        ← NEW — lookup index + hybrid resolution registry
  <category>/<name>/SKILL.md   ← ~28 skill files across bi/ data/ dbt/ eval/ collect/ generic/
```

**Collector preserves all artifact file structures unchanged.** The user-facing behavior of `/admin_resource` is preserved exactly:

| Preserved (no change) | Notes |
|---|---|
| `artifacts/CATALOG.md` | Master artifact index |
| `artifacts/CHANGELOG.md` | Artifact evolution log |
| `artifacts/.source-registry.md` | MCP server → source_type registry |
| `artifacts/versions/` | Archived prior versions |
| `artifacts/ROLLUP-current-state.md` | Rollup target |
| `artifacts/{confluence,jira,teams-chat,meeting-transcript,ad-hoc}/` | Source-typed artifact folders |
| `/admin_resource {ingest, search, status, changelog, rebuild, rollup, update}` sub-commands | Identical behavior |

Internally these are now produced by `collector` loading `collect/*` skills instead of `resource-*` agents.

### 2.3 Modify

| File | Change |
|---|---|
| `.claude/commands/project.md` | Route to `planner` only; sub-commands map to planner phases. Add playbook auto-selection from intent. |
| `.claude/commands/admin_resource.md` | Route to `collector` |
| `.claude/commands/spec.md` | Unchanged (inline command) |
| `.claude/settings.local.json` | Clean up `auto_trader`-specific absolute paths |

### 2.4 Delete

| File | Reason |
|---|---|
| `.claude/commands/bi_agent.md` | Deprecated at Phase 7. Replaced by smart routing through `/project`. Users can still force the BI playbook via `/run bi-dashboard <context>`. |

### 2.5 Add new command

| File | Purpose |
|---|---|
| `.claude/commands/run.md` | Power-user playbook runner — `/run <playbook-name> <context>`. Escape hatch only, not the recommended entry. |

---

## 3. Skill Catalog (Full List)

Total: **~28 skills** across 6 categories.

### 3.1 `skills/generic/` — cross-domain (8 skills)

| Skill | Source | Reused by |
|---|---|---|
| `requirement-intake` | `bi-requirement-intake` | BI, dbt, analysis, spec-to-build |
| `stakeholder-alignment` | `bi-stakeholder-alignment` | BI, dbt |
| `release-checklist` | `bi-release-deployment` + `bi-validation-qa` (prod-safety part) | BI, dbt |
| `documentation` | `bi-documentation-knowledge` | All playbooks |
| `governance-check` | `bi-governance-reuse` | BI, dbt |
| `source-enablement` | `bi-source-enablement` | BI, dbt |
| `journal-update` | implicit in `project-strategist` | All playbooks |
| `versioning` | implicit | All playbooks |

### 3.2 `skills/bi/` — BI-specific (4 skills)

| Skill | Source |
|---|---|
| `kpi-definition` | `bi-kpi-metric-definition` |
| `wireframe-ux` | `bi-wireframe-ux` |
| `dashboard-build` | `bi-build` |
| `bi-validation` | `bi-validation-qa` (UX + functional parts) |

### 3.3 `skills/data/` — shared data layer (3 skills)

| Skill | Source | Reused by |
|---|---|---|
| `discovery` | `bi-data-discovery` | BI, dbt, analysis |
| `quality-profiling` | `bi-data-quality-profiling` | BI, dbt |
| `semantic-modeling` | `bi-semantic-model-design` | BI, dbt |

### 3.4 `skills/dbt/` — dbt-specific (3 skills, NEW)

| Skill | Purpose |
|---|---|
| `model-build` | Build SQL/dbt models (partly from `bi-transformation-sql-build`) |
| `test-design` | dbt schema/data tests |
| `documentation` | dbt docs + lineage |

### 3.5 `skills/eval/` — evaluator (4 skills, NEW)

| Skill | Purpose |
|---|---|
| `contract-definition` | Produce a sprint contract from planner output + playbook checkpoint |
| `output-grading` | Grade generator output PASS/FAIL/PARTIAL per criterion with hard thresholds |
| `feedback-synthesis` | Convert FAIL verdict into actionable rework brief |
| `retrospective` | Post-delivery lesson-learned (from `project-reviewer`) |

### 3.6 `skills/collect/` — ingestion (6 skills)

| Skill | Source |
|---|---|
| `ingest-confluence` | `resource-ingestor` (Confluence path) |
| `ingest-jira` | `resource-ingestor` (Jira path) |
| `ingest-adhoc` | `resource-ingestor` (ad-hoc path) |
| `catalog-index` | `resource-cataloger` |
| `version-supersede` | `resource-versioner` |
| `search` | `resource-search` |
| `registry-resolve` | `resource-registry` |

---

## 4. Playbook Design

### 4.1 `playbooks/bi-dashboard.md` (sample)

```yaml
---
name: bi-dashboard
description: End-to-end BI dashboard delivery
output_root: docs/projects/<slug>/output/
---

steps:
  - id: 01-requirement
    skill: generic/requirement-intake
    output: 01-requirement/01-requirement.md
  - id: 02-kpi
    skill: bi/kpi-definition
    output: 02-kpi/02-kpi-dictionary.md
    depends_on: [01-requirement]
  - id: 03-alignment
    skill: generic/stakeholder-alignment
    output: 03-alignment/03-alignment-summary.md
    depends_on: [01-requirement, 02-kpi]
    parallel_with: [04-discovery]
  - id: 04-discovery
    skill: data/discovery
    output: 04-discovery/04-source-map.md
    depends_on: [01-requirement, 02-kpi]
    parallel_with: [03-alignment]
  # ... continue through 14

checkpoints:
  - after: [03-alignment, 04-discovery]
    eval_contract: cp1-requirements-and-feasibility
    human_gate: true
  - after: [07-sql-build]
    eval_contract: cp2-model-and-curated-data
    human_gate: true
  - after: [08-wireframe]
    eval_contract: cp3-design
    human_gate: true

cross_cutting:
  - skill: generic/source-enablement
    triggers_on: [discovery-blocked]
  - skill: generic/governance-check
    triggers_on: [kpi-defined, model-designed]
```

### 4.2 `playbooks/dbt-data-product.md` (sample)

```yaml
steps:
  - id: 01-requirement
    skill: generic/requirement-intake
  - id: 02-discovery
    skill: data/discovery
  - id: 03-profiling
    skill: data/quality-profiling
  - id: 04-model
    skill: data/semantic-modeling
  - id: 05-build
    skill: dbt/model-build
  - id: 06-test
    skill: dbt/test-design
  - id: 07-docs
    skill: dbt/documentation
  - id: 08-release
    skill: generic/release-checklist

checkpoints:
  - after: [03-profiling]
    eval_contract: cp1-feasibility
    human_gate: true
  - after: [06-test]
    eval_contract: cp2-build-quality
    human_gate: true
```

---

## 5. The Generic Loop (`rules/harness-loop.md`)

The universal pipeline every playbook runs:

```
1. collector  → artifacts in artifacts/ (optional, can pre-exist)
2. planner    → reads artifacts → picks playbook → produces strategy.md + journal
3. For each checkpoint in the chosen playbook:
   a. evaluator (contract-definition skill) → sprint-contract.md
   b. generator → runs the checkpoint's steps in order/parallel, loading skills
                  → self-checks against contract checklist before handoff
   c. evaluator (output-grading skill) → verdict.md (PASS / FAIL per criterion)
   d. If FAIL and iteration < cap:
        evaluator (feedback-synthesis skill) → feedback-<iter>.md
        loop back to (b) with feedback
   e. If FAIL and iteration == cap:
        escalate to user with full failure report (human gate)
   f. If PASS:
        human gate (user approves checkpoint)
        advance to next checkpoint
4. After all checkpoints complete:
   evaluator (retrospective skill) → lessons-learned.md
```

---

## 6. Migration Phases

Total estimated effort: **3–5 focused days**. Phases are sequential except where noted.

### Phase 0 — Foundation (½ day)

- Create `rules/` directory with all 6 invariant files
- Define handoff block format, sprint contract schema, output structure as Markdown specs
- Define `harness-loop.md` as the universal pipeline doc

**Exit criteria:** A new contributor can read `rules/` and understand the harness without reading any agents.

### Phase 1 — Generic Agents (1 day)

- Write the 4 new generic agents in `.claude/agents/`
- Keep old agents in place — do not delete yet
- Add a temporary `experimental: true` flag in front-matter to flag them as new

**Exit criteria:** Each agent file is < 200 lines, references rules/ for invariants, loads skills by path.

### Phase 2 — Evaluator Skills (½ day)

Build the eval skills first because they unblock the loop:

- `eval/contract-definition`
- `eval/output-grading`
- `eval/feedback-synthesis`
- `eval/retrospective`

**Exit criteria:** Evaluator agent + 4 skills can grade an arbitrary deliverable against an arbitrary contract.

### Phase 3 — Proof: convert ONE BI step (½ day)

Convert `bi-kpi-metric-definition.md` → `skills/bi/kpi-definition/SKILL.md`. Build minimal playbook `playbooks/kpi-only.md` with one step. Run end-to-end: planner picks playbook → generator runs skill → evaluator grades.

**Exit criteria:** A real BI KPI dictionary deliverable is produced through the new harness end-to-end. **Hard gate — if this doesn't feel right, stop and re-evaluate before Phase 4.**

### Phase 4 — Migrate remaining BI + data skills + initial catalog (1 day)

- Convert remaining BI agent files into skills under `bi/`, `data/`, `generic/`.
- Build `playbooks/bi-dashboard.md` reproducing the current 14-step flow.
- **Build initial `.claude/skills/SKILLS-CATALOG.md`** with all BI + data + generic + eval skills indexed. Include the native skills section pre-populated with relevant Claude Code plugin skills (`anthropic-skills:*`, `engineering:*`, `atlassian:*`).
- Add catalog maintenance protocol to `rules/` so every future skill change updates the catalog in the same commit.

**Exit criteria:**
- `/run bi-dashboard <context>` produces equivalent output to the old `/bi_agent` on a benchmark request
- `SKILLS-CATALOG.md` lists every skill that exists; generator can resolve any skill in one file read
- Generator never has to glob `.claude/skills/**/SKILL.md`

### Phase 5 — Migrate Collector (½ day)

- Build `collector.md` agent + ingestion skills under `collect/`.
- Rewire `/admin_resource` sub-commands to route through `collector`.
- Update `SKILLS-CATALOG.md` with the new collect skills.
- **Verify all artifact file structures are byte-identical** to old harness output on a benchmark ingest:
  - `artifacts/CATALOG.md`
  - `artifacts/CHANGELOG.md`
  - `artifacts/.source-registry.md`
  - per-source-type folders

**Exit criteria:**
- `/admin_resource ingest <url>` produces identical artifact output to the old harness
- `/admin_resource update --check`, `--supersede`, `search`, `rebuild`, `rollup` all preserved
- Catalog updated in same commit as skill creation

### Phase 6 — Build dbt playbook (½ day)

Build `playbooks/dbt-data-product.md` and the 3 new dbt skills. Run a sample dbt project request through it. **This is the real proof the harness is generic** — adding a new domain should take hours, not days.

**Exit criteria:** Working dbt model produced via `/run dbt-data-product`.

### Phase 7 — Retire old agents and deprecate `/bi_agent` (½ day)

- Delete the 25 old agent files.
- Delete `.claude/commands/bi_agent.md`.
- Remove `experimental: true` flags from the 4 new agents.
- Update `CLAUDE.md` to reflect the new command surface and architecture.
- Add a `MIGRATION-NOTES.md` to the repo root with the `/bi_agent` → `/project` deprecation note for anyone with muscle memory.
- Push to `harness_eng`.

**Exit criteria:**
- `.claude/agents/` contains exactly 4 files (`collector`, `planner`, `generator`, `evaluator`)
- `.claude/commands/` contains exactly 4 files (`project`, `admin_resource`, `spec`, `run`)
- `/project` and `/admin_resource` route through the new harness with identical (or better) results to the old harness on benchmark prompts
- `/bi_agent` returns "command not found" — verified
- `/run bi-dashboard <context>` produces equivalent output to the old `/bi_agent <context>`

---

## 7. Decisions Required From User

These need answers before Phase 0 starts.

**Resolved (2026-05-16):**

1. **Skill discovery mechanism.** ✅ **Hybrid resolution.** The generator first checks if a matching skill is available natively in the Claude Code skill system (via the plugins listed in the session, e.g. `engineering:*`, `anthropic-skills:*`, `atlassian:*`). If a native skill matches, use it. Otherwise fall back to a user-authored skill file under `.claude/skills/<category>/<name>/SKILL.md`. The playbook can specify either by path (`skill: bi/kpi-definition`) or by native name (`skill: anthropic-skills:docx`). The generator resolves at runtime.

2. **Backward compatibility window.** ✅ **Atomic switch at Phase 7.** No parallel run window. Old agents stay untouched through Phases 1–6; Phase 7 deletes them and rewires commands in a single commit.

3. **dbt scope.** ✅ **Both dbt Core and dbt Cloud.** Skills under `skills/dbt/` will be written to handle either target. Playbook passes `dbt_target: core | cloud` as context; skill branches on that.

**All resolved:**

4. **Iteration cap.** ✅ Default **5 rework iterations** per checkpoint before escalating to user. Overridable per playbook via `max_iterations:` in front-matter.

5. **Output folder structure across playbooks.** ✅ **Per-playbook layout.** Each playbook owns its output schema. BI keeps `01-14` step folders; dbt uses `models/ tests/ docs/ release/`; analysis uses `findings/ narrative/ review/`. Playbook front-matter declares `output_structure:`.

6. **Skill front-matter spec.** ✅ **YAML front-matter.** Matches Claude Code native skill convention and supports the hybrid native-or-local skill resolution from Q1. Skill files use:
   ```yaml
   ---
   name: bi/kpi-definition
   description: Convert business KPI requests into precise, gradable metric definitions
   inputs: [requirement-doc]
   outputs: [kpi-dictionary]
   model_tier_hint: sonnet
   ---
   <procedure body in Markdown>
   ```

7. **Eval grader strictness.** ✅ **PASS / FAIL only.** Hard threshold per criterion. No partial credit. If a deliverable is "mostly good but missing one thing," that's FAIL with a specific feedback item — not a soft pass.

8. **Playbook chaining.** ✅ **Single-playbook per invocation.** Each `/run` or `/project` runs exactly one playbook. To chain workflows (e.g., dbt model → BI dashboard), the user invokes twice. Keeps state, gates, and rollback simple.

---

## 8. Risks & Mitigations

| Risk | Mitigation |
|---|---|
| Generic generator drifts without strict skill files | Phase 3 hard gate. If drift is observed, push more constraints into SKILL.md (templates, examples, checklists). |
| Eval grader rubber-stamps work (over-praises) | Tune `eval/output-grading` SKILL.md with explicit skepticism instructions, examples of failed work, "default to FAIL unless clearly meets all criteria" framing. |
| Migration breaks active projects | All work happens in `experimental: true` mode; old agents remain until Phase 7. Use a feature flag in commands to switch routing. |
| Skill files balloon and become as bloated as old agents | Set a soft cap (~300 lines / skill). If exceeded, split into sub-skills. |
| Loss of model tier optimization | Agent-level model tier still applies (collector=Sonnet, planner=Opus, generator=Sonnet, evaluator=Opus). Skills inherit the calling agent's tier. |
| Playbook YAML grows brittle | Keep playbooks in Markdown with structured YAML front-matter for sequencing, prose for cross-cutting guidance. Validate playbook structure in `rules/`. |
| Cross-playbook skill conflicts (BI vs. dbt expect different conventions in `semantic-modeling`) | Skills are *opinionated but parameterizable* — playbooks pass context (e.g., `target: dbt` vs `target: powerbi-dataset`) into the skill invocation. |

---

## 9. Acceptance Criteria for Completed Migration

The migration is "done" when **all** of the following are true:

1. `.claude/agents/` contains exactly 4 files: `collector`, `planner`, `generator`, `evaluator`.
2. A canonical BI dashboard request flows through the new harness end-to-end and produces deliverables of equivalent quality to the old harness on a benchmark prompt.
3. A dbt data product request runs successfully through `/run dbt-data-product`.
4. Adding a hypothetical third playbook (e.g., `analysis-deep-dive`) is a **single PR** that touches one playbook file and 0–2 new skills — no agent changes.
5. The eval loop demonstrably catches at least one quality regression on a deliberately-broken test deliverable.
6. `CLAUDE.md` is updated to describe the new architecture.
7. All `rules/` files are present and self-consistent.
8. `harness_eng` GitHub repo has the new structure on `main` and the old structure tagged as `v0-domain-specific` for rollback.

---

## 10. Open Questions to Resolve During Execution

These don't block approval but will need answers as we go:

- Should `collector` be a single agent or split into "ingest" and "search" agents? (Article would say single — load skills.)
- Should the planner produce the sprint contract itself, or always defer to evaluator? (Cleaner: always evaluator owns contracts.)
- How does the human gate notify the user? (Currently implicit — should be explicit in `rules/harness-loop.md`.)
- Versioning of skills themselves — do skills get the universal versioning protocol applied to them, or are they treated like code (git-versioned only)?

---

## 11. What This Plan Deliberately Does NOT Do

- Does not add MCP integrations beyond what already exists.
- Does not change the universal versioning protocol — that survives unchanged.
- Does not introduce a database or non-Markdown state.
- Does not change the handoff block convention.
- Does not change how `/spec` works.
- Does not introduce parallel agent execution beyond what playbooks declare.

---

**End of plan. Awaiting your review and decisions on Section 7 before execution.**
