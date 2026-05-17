# Lessons Library

Distilled anti-pattern notes that apply across multiple skills. Each lesson has a stable `lesson_id` (`L-NNN`) that skills can reference by citation rather than duplicating the rule inline.

## Why this exists

When the same failure mode keeps surfacing across different skills, copy-pasting the rule into every skill creates drift — one update misses some copies. Instead, write the rule once here and have each skill cite it.

## File naming

```
L-<NNN>-<short-slug>.md
```

NNN is a globally unique 3-digit zero-padded sequence number across the entire harness (lessons in this folder + `## Lessons Learned` sections embedded in SKILL.md files all share one ID space).

## How a skill references a lesson

In a self-check item or step list:
```markdown
- [ ] No field contains TBD as its only content (see lessons/L-001)
```

The rule itself must still be stated in the skill — the reference signals provenance and provides deeper context for anyone investigating.

## Lifecycle

| State | What it means |
|---|---|
| `active` | Currently enforced; meta-learner watches whether it still applies |
| `superseded` | Replaced by a newer lesson; preserved for history (never deleted) |

Marking a lesson superseded requires a meta-learner proposal + human approval.

## Index

| ID | Title | Status | Applies to |
|---|---|---|---|
| L-001 | TBD as sole field content is never acceptable | active | all skills with field-content deliverables |

See `.claude/rules/lessons-learned-protocol.md` for full schema and authoring rules.
