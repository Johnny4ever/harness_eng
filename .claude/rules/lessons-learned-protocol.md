# Lessons Learned Protocol

This protocol governs how the harness records *why* a skill, agent, or rule is the way it is. Two complementary mechanisms:

1. **Embedded `## Lessons Learned` section** in each SKILL.md (and selected agent/rule files) — local institutional memory
2. **`.claude/lessons/` library** — distilled cross-cutting anti-patterns with stable IDs that skills can reference

## When to Record a Lesson

Record a lesson when one of these is true:

| Trigger | Example |
|---|---|
| Evaluator caught a recurring failure mode that the skill's current instructions allowed | TBD-as-only-content bug (L-001) |
| A real project surfaced a gotcha that future projects should not re-discover | Event-vs-snapshot grain disambiguation (L-007) |
| A stakeholder corrected a misunderstanding that was encoded in skill behaviour | "Trial conversion is calendar-month, not 30-day window" |
| A proposal from the meta-learner was accepted and applied | All accepted proposals create a corresponding lesson |

**Do NOT** record a lesson for:
- One-off mistakes with no recurrence pattern
- Project-specific quirks (those stay in `project-journal.md`)
- Mistakes the existing skill instructions already cover

## `## Lessons Learned` Section in SKILL.md

Every skill file ends with this section (added in Phase 8.0 to all existing skills as an empty placeholder):

```markdown
## Lessons Learned

<!-- 
Append one bullet per lesson. Newest at the top.
Format:
- **<YYYY-MM-DD> — <short title>.**
  Trigger: <1-sentence — which project(s) and what failure pattern>.
  Change: <1-sentence — what was modified>.
  learning_id: L-NNN
-->
```

### Field rules

| Field | Rule |
|---|---|
| Date | Date the lesson was *recorded*, not the date of the triggering failure |
| Short title | Imperative or descriptive phrase, under 60 chars |
| Trigger | Cite project slug(s) and the failure pattern in plain English |
| Change | What concretely changed in the skill (a step added, a self-check item tightened, etc.) |
| `learning_id` | Always assigned; format `L-NNN` zero-padded 3-digit. See ID assignment below |

### ID assignment

`learning_id` is a globally unique counter across the entire harness, not per-skill.

To get the next ID:
1. Scan `.claude/lessons/` for the highest `L-NNN` filename
2. Scan all `## Lessons Learned` sections for the highest `learning_id`
3. Take max + 1

Implementation: the meta-learner does this scan when drafting a proposal that involves a new lesson; the human applying the change uses the ID the proposal specified.

## `.claude/lessons/` Library

When a lesson applies to **more than one skill**, distill it into a standalone lesson file. This keeps each SKILL.md short and prevents duplicated wording.

### File path
```
.claude/lessons/L-<NNN>-<slug>.md
```
Slug: 3–6 lowercase hyphenated words capturing the essence. Example: `L-001-tbd-as-sole-content.md`

### Lesson file schema
```markdown
---
lesson_id: L-NNN
title: <full title>
created: <YYYY-MM-DD>
applies_to: [list of skills, agents, or "all"]
status: active | superseded
superseded_by: L-NNN | null
trigger_projects: [<project-slug-1>, <project-slug-2>, ...]
---

# L-NNN: <title>

## The trap
<2–4 sentence description of the failure mode>

## The rule
<2–4 sentence statement of what must be true to avoid the trap>

## How to reference
<Show how a SKILL.md self-check or step should cite this lesson>

## History
- <YYYY-MM-DD>: Recorded. Source: <project slug, proposal_id if applicable>
- <YYYY-MM-DD>: Verified active. <N> invocations since application, <N> first-pass PASS.
```

### Referencing a lesson from a skill

In a skill's self-check or step list:
```markdown
- [ ] No field contains TBD as its only content (see lessons/L-001)
```

The reference is informational — the rule itself must still be stated in the skill. The reference signals "this rule comes from a recorded lesson, see context."

## Lesson Lifecycle

| State | Meaning |
|---|---|
| `active` | Currently enforced; meta-learner monitors continued relevance |
| `superseded` | Replaced by a newer lesson; kept for historical reference |

A lesson is superseded (not deleted) when:
- A better-stated lesson covers the same ground
- The model has improved enough that the original trap no longer applies
- A change in conventions has made the rule moot

Marking superseded requires a meta-learner proposal and human approval — same gating as creating a new lesson.

## Anti-Patterns

| Anti-pattern | Why it's bad |
|---|---|
| Writing a lesson for a one-off mistake | Lesson library becomes noise; skills bloat with rare cases |
| Inlining a multi-skill lesson into every skill | Drift across copies; one update misses some |
| Deleting a lesson when it stops applying | Loses the history of *why* the harness once worked that way |
| Vague lesson rules ("be careful with dates") | Cannot be enforced or evaluated; not a real rule |
| Lessons that contradict an existing lesson without superseding it | Skill behaviour becomes ambiguous |

## Self-Check (for the human or meta-learner applying a lesson)

- [ ] Lesson has a unique `learning_id` (no collision with existing IDs)
- [ ] `applies_to` lists every affected skill/agent/rule
- [ ] "The rule" section is concrete enough to evaluate (PASS/FAIL on observation)
- [ ] Trigger project(s) are named — no "we noticed in some projects"
- [ ] If shared across skills, lives in `.claude/lessons/` (not duplicated inline)
- [ ] The corresponding `## Lessons Learned` entry exists in every skill in `applies_to`
